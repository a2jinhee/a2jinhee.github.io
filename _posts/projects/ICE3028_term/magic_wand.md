---
layout: default
title: Magic Wand Github Code Review
nav_exclude: true
grand_parent: Projects
parent: ICE3028 Term Project
show_toc: true 
---

## Magic Wand Github Code Review  
{: .no_toc }

_2023 10.29_  
<br>

*reference*  

- github REPO: [magic_wand](https://github.com/petewarden/magic_wand)

--- 

1. TOC
{:toc}

--- 

## 1. magic_wand.ino

### [1] setup()    

```cpp
void setup() {
    Serial.begin(9600); //baudrate(bits/sec) 9600
    Serial.println("Started");

    /* IMU setup */
    if (!IMU.begin()) {
        Serial.println("Failed to initialized IMU!");
        while (1);
    }
    SetupIMU();

    /* BLE setup */ 
    if (!BLE.begin()) {
        Serial.println("Failed to initialized BLE!");
        while (1);
    }

    String address = BLE.address();
    Serial.print("address = ");
    Serial.println(address);
    address.toUpperCase();

    name = "BLESense-";
    name += address[address.length() - 5];
    name += address[address.length() - 4];
    name += address[address.length() - 2];
    name += address[address.length() - 1];

    Serial.print("name = ");
    Serial.println(name);

    BLE.setLocalName(name.c_str());
    BLE.setDeviceName(name.c_str());
    BLE.setAdvertisedService(service);
    service.addCharacteristic(strokeCharacteristic);
    BLE.addService(service);
    BLE.advertise();

    /* Error reporter for TFLite inference code */
    // NOLINT => suppress warnings raised by the tool
    static tflite::MicroErrorReporter micro_error_reporter; // NOLINT
    error_reporter = &micro_error_reporter;

    /* Moel mapping to usable data structure. */
    // g_magic_wand_model_data => TFLite model .bin
    model = tflite::GetModel(g_magic_wand_model_data);
    if (model->version() != TFLITE_SCHEMA_VERSION) {
        TF_LITE_REPORT_ERROR(error_reporter,
                            "Model provided is schema version %d not equal "
                            "to supported version %d.",
                            model->version(), TFLITE_SCHEMA_VERSION);
        return;
    }

    /* Operation Init */
    // Pull in only needed ops from the graph 
    // 'AllOpsResolver' is viable, but incurs penalty in code space
    static tflite::MicroMutableOpResolver<4> micro_op_resolver;  // NOLINT
    micro_op_resolver.AddConv2D();
    micro_op_resolver.AddMean();
    micro_op_resolver.AddFullyConnected();
    micro_op_resolver.AddSoftmax();

    /* Build Interpreter */
    // Configuring a TFLite interpreter to run    
    // a TFLite model on an embedded system (microcontroller) 
    static tflite::MicroInterpreter static_interpreter(
        model, micro_op_resolver, tensor_arena, kTensorArenaSize, error_reporter);
    interpreter = &static_interpreter;

    /* Memory management */
    // Allocate memory from the tensor_arena for the model's tensors.
    interpreter->AllocateTensors();

    /* Validates input tensor parameters */
    // Checks if the input tensor has the expected dim. and data type
    TfLiteTensor* model_input = interpreter->input(0);
    if ((model_input->dims->size != 4) || (model_input->dims->data[0] != 1) ||
        (model_input->dims->data[1] != raster_height) ||
        (model_input->dims->data[2] != raster_width) ||
        (model_input->dims->data[3] != raster_channels) ||
        (model_input->type != kTfLiteInt8)) {
        TF_LITE_REPORT_ERROR(error_reporter,
                            "Bad input tensor parameters in model");
        return;
    }

    /* Validates output tensor parameters */
    // Checks if the output tensor has the expected dim. and data type
    TfLiteTensor* model_output = interpreter->output(0);
    if ((model_output->dims->size != 2) || (model_output->dims->data[0] != 1) ||
        (model_output->dims->data[1] != label_count) ||
        (model_output->type != kTfLiteInt8)) {
        TF_LITE_REPORT_ERROR(error_reporter,
                            "Bad output tensor parameters in model");
        return;
    }

}
```


### [2] loop()  
```cpp
void loop() {
    /* Attempt to connect to a central BLE device */
    BLEDevice central = BLE.central();
    
    /* Check if central device is connected */
    static bool was_connected_last = false;  
    if (central && !was_connected_last) {
        Serial.print("Connected to central: ");
        Serial.println(central.address()); //Print central BT address
    }
    was_connected_last = central;
    
    /* Check if accelerometor or gyrosocope is available */
    const bool data_available = IMU.accelerationAvailable() || IMU.gyroscopeAvailable();
    if (!data_available) {
        return;
    }

    /* Read accelerometor & gyrosocope data */
    int accelerometer_samples_read;
    int gyroscope_samples_read;
    ReadAccelerometerAndGyroscope(&accelerometer_samples_read, &gyroscope_samples_read);

    /* Process gyrosocope data */
    bool done_just_triggered = false;
    if (gyroscope_samples_read > 0) {
        EstimateGyroscopeDrift(current_gyroscope_drift);
        UpdateOrientation(gyroscope_samples_read, current_gravity, current_gyroscope_drift);
        UpdateStroke(gyroscope_samples_read, &done_just_triggered);
        // If a central device is connected, send the stroke data
        if (central && central.connected()) {
            strokeCharacteristic.writeValue(stroke_struct_buffer, stroke_struct_byte_count);
        }
    }

    /* Process accelerometer data */
    if (accelerometer_samples_read > 0) {
        EstimateGravityDirection(current_gravity);
        UpdateVelocity(accelerometer_samples_read, current_gravity);
    }

    /* If stroke event triggered, rasterize stroke and print */
    if (done_just_triggered) {
        /* Rasterize stroke */
        RasterizeStroke(stroke_points, *stroke_transmit_length, 0.6f, 0.6f, raster_width, raster_height, raster_buffer);

        /* Print stroke */
        for (int y = 0; y < raster_height; ++y) {
        char line[raster_width + 1];
        for (int x = 0; x < raster_width; ++x) {
            const int8_t* pixel = &raster_buffer[(y * raster_width * raster_channels) + (x * raster_channels)];
            const int8_t red = pixel[0];
            const int8_t green = pixel[1];
            const int8_t blue = pixel[2];
            char output;
            if ((red > -128) || (green > -128) || (blue > -128)) {
            output = '#';
            } else {
            output = '.';
            }
            line[x] = output;
        }
        line[raster_width] = 0;
        Serial.println(line);
        }
        
        /* Prepare and run inference using TFLite model */
        // Model input <= 'raster_buffer'
        TfLiteTensor* model_input = interpreter->input(0);
        for (int i = 0; i < raster_byte_count; ++i) {
        model_input->data.int8[i] = raster_buffer[i];
        }

        // Check for errors 
        TfLiteStatus invoke_status = interpreter->Invoke();
        if (invoke_status != kTfLiteOk) {
        TF_LITE_REPORT_ERROR(error_reporter, "Invoke failed");
        return;
        }

        // Retrieve output tensor from the TFLite model
        TfLiteTensor* output = interpreter->output(0);

        /* Find class with highest score */
        int8_t max_score;
        int max_index;
        for (int i = 0; i < 10; ++i) {
            const int8_t score = output->data.int8[i];
            if ((i == 0) || (score > max_score)) {
                max_score = score;
                max_index = i;
            }
        }
        TF_LITE_REPORT_ERROR(error_reporter, "Found %s (%d)", labels[max_index], max_score);
    }
}

```

### [3] TFLite Interpreter 공부

## 2. train_magic_wand_model.ipynb

### [1] Dataset Loader

```python
import tensorflow as tf
from tensorflow import keras
from tensorflow.keras.utils import image_dataset_from_directory

validation_ds = image_dataset_from_directory(
    directory='validation',
    labels='inferred',
    label_mode='categorical',
    batch_size=32,
    image_size=(IMAGE_WIDTH, IMAGE_HEIGHT)).prefetch(buffer_size=32)

train_ds = image_dataset_from_directory(
    directory='train',
    labels='inferred',
    label_mode='categorical',
    batch_size=32,
    image_size=(IMAGE_WIDTH, IMAGE_HEIGHT)).prefetch(buffer_size=32)

```

### [2] Model Layers

```python
from keras import layers

def make_model(input_shape, num_classes):
    inputs = keras.Input(shape=input_shape)

    # Entry block
    x = layers.Rescaling(1.0 / 255)(inputs)
    x = layers.Conv2D(16, 3, strides=2, padding="same")(x)
    x = layers.BatchNormalization()(x)
    x = layers.Activation("relu")(x)
    x = layers.Dropout(0.5)(x)

    x = layers.Conv2D(32, 3, strides=2, padding="same")(x)
    x = layers.BatchNormalization()(x)
    x = layers.Activation("relu")(x)
    x = layers.Dropout(0.5)(x)

    x = layers.Conv2D(64, 3, strides=2, padding="same")(x)
    x = layers.BatchNormalization()(x)
    x = layers.Activation("relu")(x)
    x = layers.Dropout(0.5)(x)

    x = layers.GlobalAveragePooling2D()(x)
    activation = "softmax"
    units = num_classes

    x = layers.Dropout(0.5)(x)
    outputs = layers.Dense(units, activation=activation)(x)
    return keras.Model(inputs, outputs)

```

### [3] Training 

```python
epochs = 30

callbacks = [
    keras.callbacks.ModelCheckpoint("checkpoints/save_at_{epoch}.h5"),
]
# Set optimizer and loss function
model.compile(
    optimizer=keras.optimizers.Adam(1e-3),
    loss="binary_crossentropy",
    metrics=["accuracy"],
)
# Training 
model.fit(
    train_ds, epochs=epochs, callbacks=callbacks, validation_data=validation_ds,
)
```

### [4] Predict 

```python
def predict_image(model, filename):
  img = keras.preprocessing.image.load_img(filename, target_size=(IMAGE_WIDTH, IMAGE_HEIGHT))
  img_array = keras.preprocessing.image.img_to_array(img)
  img_array = tf.expand_dims(img_array, 0)  # Create batch axis
  predictions = model.predict(img_array).flatten()
  predicted_label_index = np.argmax(predictions)
  predicted_score = predictions[predicted_label_index]
  return (predicted_label_index, predicted_score)
  
index, score = predict_image(model, "test/7/2.png")

print(index, score)
```

### [5] Test 

```python
from IPython.display import Image, display

SCORE_THRESHOLD = 0.75

correct_count = 0
wrong_count = 0
discarded_count = 0
for label_dir in glob.glob("test/*"):
  label = int(label_dir.replace("test/", ""))
  for filename in glob.glob(label_dir + "/*.png"):
    index, score = predict_image(model, filename)
    if score < SCORE_THRESHOLD:
      discarded_count += 1
      continue
    if index == label:
      correct_count += 1
    else:
      wrong_count += 1
      print("%d expected, %d found with score %f" % (label, index, score))
      display(Image(filename=filename))

correct_percentage = (correct_count / (correct_count + wrong_count)) * 100
print("%.1f%% correct (N=%d, %d unknown)" % (correct_percentage, (correct_count + wrong_count), discarded_count))
```
-> test dataset 어디에 있는지는 잘 모르겠음

### [6] TFLite Converter - Model Quantization (⭐️)

```python
#######################################
# TFLite Converter Initialization
#######################################
converter = tf.lite.TFLiteConverter.from_saved_model(SAVED_MODEL_FILENAME)
# Convert model to a floating-point TFLite model
model_no_quant_tflite = converter.convert()
open(FLOAT_TFL_MODEL_FILENAME, "wb").write(model_no_quant_tflite)

#######################################
# Representative Dataset Function
# : represents the input data distribution
#   used to calibrate the model 
#######################################
def representative_dataset():
  for filename in glob.glob("test/*/*.png"):
    img = keras.preprocessing.image.load_img(filename, target_size=(IMAGE_WIDTH, IMAGE_HEIGHT))
    img_array = keras.preprocessing.image.img_to_array(img)
    img_array = tf.expand_dims(img_array, 0)  # Create batch axis      for images, labels in train_ds.take(1):
    yield([img_array])

#######################################
# Quantization Configuration - integer only 
#######################################
converter.optimizations = [tf.lite.Optimize.DEFAULT]
converter.target_spec.supported_ops = [tf.lite.OpsSet.TFLITE_BUILTINS_INT8]
converter.inference_input_type = tf.int8
converter.inference_output_type = tf.int8

#######################################
# Quantization using representative ds
#######################################
converter.representative_dataset = representative_dataset
model_tflite = converter.convert()
open(QUANTIZED_TFL_MODEL_FILENAME, "wb").write(model_tflite)
```
- saved_model -> float_model.tfl: use TFLite converter
- float_model.tfl -> quantized_model.tfl: use representative dataset + TFLite converter

### [7] TFLite Predict - Running inference on a TFLite model

```python
def predict_tflite(tflite_model, filename):

    # Load and prepare input image 
    img = keras.preprocessing.image.load_img(filename, target_size=(IMAGE_WIDTH, IMAGE_HEIGHT))
    img_array = keras.preprocessing.image.img_to_array(img)
    img_array = tf.expand_dims(img_array, 0)

    # Initialize the TFLite interpreter
    interpreter = tf.lite.Interpreter(model_content=tflite_model)
    interpreter.allocate_tensors()

    input_details = interpreter.get_input_details()[0]
    output_details = interpreter.get_output_details()[0]

    # If required, quantize the input layer (from float to integer)
    input_scale, input_zero_point = input_details["quantization"]
    if (input_scale, input_zero_point) != (0.0, 0):
    img_array = np.multiply(img_array, 1.0 / input_scale) + input_zero_point
    img_array = img_array.astype(input_details["dtype"])

    #######################################
    # Invoke the interpreter
    # : Invokes the TFLite interpreter to run inference on input
    #######################################
    interpreter.set_tensor(input_details["index"], img_array)
    interpreter.invoke()
    pred = interpreter.get_tensor(output_details["index"])[0]

    # If required, dequantized the output layer (from integer to float)
    output_scale, output_zero_point = output_details["quantization"]
    if (output_scale, output_zero_point) != (0.0, 0):
    pred = pred.astype(np.float32)
    pred = np.multiply((pred - output_zero_point), output_scale)

    # Determine the predicted label and score
    predicted_label_index = np.argmax(pred)
    predicted_score = pred[predicted_label_index]
    return (predicted_label_index, predicted_score)
```