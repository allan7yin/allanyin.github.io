### Multi-Modal Deep Learning

The primary goal of Multi-modal Deep Learning is to create a shared representation space that can effectively capture complementary information from different modalities. This shared representation can then be used to perform various tasks, such as image captioning, speech recognition, and natural language processing. Multi-modal Deep Learning models typically consist of multiple [neural networks](https://blog.roboflow.com/what-is-a-neural-network/), each specialized in analyzing a particular modality. The output of these networks is then combined using various fusion techniques, such as early fusion, late fusion, or hybrid fusion, to create a joint representation of the data.

  

Multi-modal deep learning models are typically composed of multiple uni-modal neural networks, which process each input modality separately. For instance, an audiovisual model may have two uni-modal networks, one for audio and another for visual data. This individual processing of each modality is known as [encoding](https://blog.roboflow.com/what-is-an-autoencoder-computer-vision/). Once uni-modal encoding is done, the information extracted from each modality must be integrated or fused. There are several fusion techniques available, ranging from simple concatenation to attention mechanisms. Multi-modal data fusion is a critical factor for the success of these  
models. Finally, a "decision" network accepts the fused encoded information and is trained on the task at hand.  

  

In general, multi-modal architectures consist of three parts:

1. Uni-modal encoders encode individual modalities. Usually, one for each input modality.
2. A fusion network that combines the features extracted from each input modality, during the encoding phase.
3. A classifier that accepts the fused data and makes predictions.

![[Screenshot_2024-02-19_at_10.29.34_PM.png]]

### Note:

The notebook can be found here, for the source code:

> [!info] Google Colaboratory  
>  
> [https://colab.research.google.com/drive/1ydCvpgq6CelqG8NVeKHBlPiU57nkr61I#scrollTo=eIRkU1VUl93c](https://colab.research.google.com/drive/1ydCvpgq6CelqG8NVeKHBlPiU57nkr61I#scrollTo=eIRkU1VUl93c)  

### `Config`

A very handy practice by the `keras` team is the use of a `config` class. With this, we can organize a lot of parameters that we may want to change, but are used in critical sections of the code base. Things like this include `epochs`, `seed`, and `image paths`:

```Python
class Config:
    sub = "sample_submission.csv"
    trgts = "target_name_meta.tsv"
    train_path = "train.csv"
    test_path = "test.csv"
    train_image_path = "train_images/"
    test_image_path = "test_images/"
    seed = 42
    fold = 0 # which fold to set as validation data
    image_size = [512, 512]
    batch_size = 96
    epochs = 10
    num_classes = 6
    num_folds = 5
    class_names = ['X4_mean', 'X11_mean', 'X18_mean',
                   'X26_mean', 'X50_mean', 'X3112_mean',]
    aux_class_names = list(map(lambda x: x.replace("mean","sd"), class_names))
    aux_num_classes = len(aux_class_names)

config = Config()
```

### Data Pre-Processing

Its better to build functions for processing data. Instead of building the data pipeline like before (scaling all the training data, then mapping them to labels, decoding images into tensors) where we are writing processing code for both training, validation, and test data, its better to build re-usable modules for this:

```Python
# responsible for data augmentation -> let this be default behaviour
def build_augmenter():
    # Define augmentations -> applied to each image after one forward pass
    aug_layers = [
        keras_cv.layers.RandomBrightness(factor=0.1, value_range=(0, 1)),
        keras_cv.layers.RandomContrast(factor=0.1, value_range=(0, 1)),
        keras_cv.layers.RandomSaturation(factor=(0.45, 0.55)),
        keras_cv.layers.RandomHue(factor=0.1, value_range=(0, 1)),
        keras_cv.layers.RandomCutout(height_factor=(0.06, 0.15), width_factor=(0.06, 0.15)),
        keras_cv.layers.RandomFlip(mode="horizontal_and_vertical"),
        keras_cv.layers.RandomZoom(height_factor=(0.05, 0.15)),
        keras_cv.layers.RandomRotation(factor=(0.01, 0.05)),
    ]

    # Apply augmentations to random samples
    aug_layers = [keras_cv.layers.RandomApply(x, rate=0.5) for x in aug_layers]

    # Build augmentation layer
    augmenter = keras_cv.layers.Augmenter(aug_layers)

    # Apply augmentations
    def augment(input, label):
        images = input["images"]
        aug_data = {"images": images}
        aug_data = augmenter(aug_data)
        input["images"] = aug_data["images"]
        return input, label
    return augment

# resonsible for converting images into arrays
def build_decoder(with_labels=True, target_size=config.image_size):
    def decode_image(input):
        path = input["images"]

        # Read jpeg image
        file_bytes = tf.io.read_file(path)
        image = tf.io.decode_jpeg(file_bytes)

        # Resize
        image = tf.image.resize(image, size=target_size, method="area")

        # Rescale image
        image = tf.cast(image, tf.float32)
        image /= 255.0

        # Reshape
        image = tf.reshape(image, [*target_size, 3])

        input["images"] = image
        return input

    def decode_label(label, num_classes):
        label = tf.cast(label, tf.float32)
        label = tf.reshape(label, [num_classes])
        return label

    def decode_with_labels(input, labels=None):
        input = decode_image(input)
        label = decode_label(labels[0], config.num_classes)
        aux_label = decode_label(labels[1], config.aux_num_classes)
        return (input, (label, aux_label))

    return decode_with_labels if with_labels else decode_image


def build_dataset(
    paths,
    features,
    labels=None,
    aux_labels=None,
    batch_size=32,
    cache=True,
    decode_fn=None,
    augment_fn=None,
    augment=False,
    repeat=True,
    shuffle=1024,
    cache_dir="",
    drop_remainder=False,
):
    if cache_dir != "" and cache is True:
        os.makedirs(cache_dir, exist_ok=True)
    
    if decode_fn is None:
        decode_fn = build_decoder(labels is not None or aux_labels is not None)

    if augment_fn is None:
        augment_fn = build_augmenter()

    AUTO = tf.data.experimental.AUTOTUNE

    input = {"images": paths, "features": features}
    slices = (input, (labels, aux_labels)) if labels is not None else inp
      

    ds = tf.data.Dataset.from_tensor_slices(slices)
    ds = ds.map(decode_fn, num_parallel_calls=AUTO)
    ds = ds.cache(cache_dir) if cache else ds
    ds = ds.repeat() if repeat else ds
    if shuffle:
        ds = ds.shuffle(shuffle, seed=config.seed)
        opt = tf.data.Options()
        opt.experimental_deterministic = False
        ds = ds.with_options(opt)
    ds = ds.batch(batch_size, drop_remainder=drop_remainder)
    ds = ds.map(augment_fn, num_parallel_calls=AUTO) if augment else ds
    ds = ds.prefetch(AUTO)
    return ds
```

Also, instead of clogging up our model definition with the various data augmentations we will apply, its better to create a separate augmentation pipeline. `keras_cv` provides us with this option. This is better than what you are used to (adding data augmentation as part of the model definition).

### Discretizing Continuous Data For Data Split

In this plant trait prediction task, we are given ancillary data in the form of tabular data (climate data). We are interested in 6 classes (which are columns in this data). When we split our data for training and validation, we need to produce a split, where the respective proportion of classes is the same in both groups.

```Python
from sklearn.model_selection import StratifiedKFold

# create this object to split the dataset later
skf = StratifiedKFold(n_splits=config.num_folds, shuffle=True, random_state=42)

# Create separate bin for each traits
for i, trait in enumerate(config.class_names):

    # Determine the bin edges dynamically based on the distribution of traits
    bin_edges = np.percentile(df[trait], np.linspace(0, 100, config.num_folds + 1))
    df[f"bin_{i}"] = np.digitize(df[trait], bin_edges)

# Concatenate the bins into a final bin
df["final_bin"] = (
    df[[f"bin_{i}" for i in range(len(config.class_names))]]
    .astype(str)
    .agg("".join, axis=1)
)

# Perform the stratified split using final bin
df = df.reset_index(drop=True)
for fold, (train_idx, valid_idx) in enumerate(skf.split(df, df["final_bin"])):
    df.loc[valid_idx, "fold"] = fold
```

We create bins for each classes we need an even spread of. Once we have this, we concatenate the 6 classes “bin” into one string, and split the data by that parameter using `stratifiedKFold`.

### Model

The most important part of all this is the model. We are building a multi-modal model, that takes in image data as well as numerical data. The `keras` team’s first notebook was:

```Python
# Define input layers
img_input = keras.Input(shape=(*CFG.image_size, 3), name="images")
feat_input = keras.Input(shape=(len(FEATURE_COLS),), name="features")

# Branch for image input
backbone = keras_cv.models.EfficientNetV2Backbone.from_preset(CFG.preset)
x1 = backbone(img_input)
x1 = keras.layers.GlobalAveragePooling2D()(x1)
x1 = keras.layers.Dropout(0.2)(x1)

# Branch for tabular/feature input
x2 = keras.layers.Dense(326, activation="selu")(feat_input)
x2 = keras.layers.Dense(64, activation="selu")(x2)
x2 = keras.layers.Dropout(0.1)(x2)

# Concatenate both branches
concat = keras.layers.Concatenate()([x1, x2])

# Output layer
out1 = keras.layers.Dense(CFG.num_classes, activation=None, name="head")(concat)
out2 = keras.layers.Dense(CFG.aux_num_classes, activation="relu", name="aux_head")(concat)
out = {"head": out1, "aux_head":out2}

# Build model
model = keras.models.Model([img_input, feat_input], out)

# Compile the model
model.compile(
    optimizer=keras.optimizers.Adam(learning_rate=1e-4),
    loss={
        "head": R2Loss(use_mask=False),
        "aux_head": R2Loss(use_mask=True), # use_mask to ignore `NaN` auxiliary labels
    },
    loss_weights={"head": 1.0, "aux_head": 0.3},  # more weight to main task
    metrics={"head": R2Metric()}, # evaluation metric only on main task
)

# Model Summary
model.summary()
```

![[Untitled.png]]

Here is what the model looks like. After running `model.summary()`, we see:

```Python
Model: "PlantTraits2024"

┏━━━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ Layer (type)              ┃ Output Shape           ┃        Param # ┃ Connected to           ┃
┡━━━━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━┩
│ images (InputLayer)       │ (None, 512, 512, 3)    │              0 │ -                      │
├───────────────────────────┼────────────────────────┼────────────────┼────────────────────────┤
│ features (InputLayer)     │ (None, 163)            │              0 │ -                      │
├───────────────────────────┼────────────────────────┼────────────────┼────────────────────────┤
│ efficient_net_v2b2_backb… │ (None, 16, 16, 1408)   │      8,769,374 │ images[0][0]           │
│ (EfficientNetV2Backbone)  │                        │                │                        │
├───────────────────────────┼────────────────────────┼────────────────┼────────────────────────┤
│ dense (Dense)             │ (None, 326)            │         53,464 │ features[0][0]         │
├───────────────────────────┼────────────────────────┼────────────────┼────────────────────────┤
│ global_average_pooling2d  │ (None, 1408)           │              0 │ efficient_net_v2b2_ba… │
│ (GlobalAveragePooling2D)  │                        │                │                        │
├───────────────────────────┼────────────────────────┼────────────────┼────────────────────────┤
│ dense_1 (Dense)           │ (None, 64)             │         20,928 │ dense[0][0]            │
├───────────────────────────┼────────────────────────┼────────────────┼────────────────────────┤
│ dropout (Dropout)         │ (None, 1408)           │              0 │ global_average_poolin… │
├───────────────────────────┼────────────────────────┼────────────────┼────────────────────────┤
│ dropout_1 (Dropout)       │ (None, 64)             │              0 │ dense_1[0][0]          │
├───────────────────────────┼────────────────────────┼────────────────┼────────────────────────┤
│ concatenate (Concatenate) │ (None, 1472)           │              0 │ dropout[0][0],         │
│                           │                        │                │ dropout_1[0][0]        │
├───────────────────────────┼────────────────────────┼────────────────┼────────────────────────┤
│ aux_head (Dense)          │ (None, 6)              │          8,838 │ concatenate[0][0]      │
├───────────────────────────┼────────────────────────┼────────────────┼────────────────────────┤
│ head (Dense)              │ (None, 6)              │          8,838 │ concatenate[0][0]      │
└───────────────────────────┴────────────────────────┴────────────────┴────────────────────────┘

 Total params: 8,861,442 (33.80 MB)

 Trainable params: 8,779,154 (33.49 MB)

 Non-trainable params: 82,288 (321.44 KB)
```

This is the basic approach, which will yield less accurate predictions. To improve upon this, we will need to add additional convolutional layers after the model base. In particular, we want to use the `conv base` to `predict()` the input data, instead of training on it. We are seeing very long train times since currently which is due to the fat that the model parameters of the pre-trained model are also being updated. We need to fix this and make the base `untrainable`.

![[Screenshot_2024-02-20_at_11.18.37_PM.png]]

The above is the performance after we made the following change:

```Python
# Branch for image input
backbone = keras_cv.models.EfficientNetV2Backbone.from_preset(CFG.preset)
# Freeze the backbone layers
for layer in backbone.layers:
    layer.trainable = False
    
x1 = backbone(img_input)
x1 = keras.layers.BatchNormalization()(x1)
x1 = keras.layers.SeparableConv2D(filters=1408, kernel_size=3, padding="same", use_bias=False)(x1)
x1 = keras.layers.BatchNormalization()(x1)
x1 = keras.layers.Activation("relu")(x1)
x1 = keras.layers.SeparableConv2D(filters=1408, kernel_size=3, padding="same", use_bias=False)(x1)
x1 = keras.layers.GlobalAveragePooling2D()(x1)
x1 = keras.layers.Dropout(0.2)(x1)

# Branch for tabular/feature input
x2 = keras.layers.Dense(326, activation="selu")(feat_input)
x2 = keras.layers.Dense(256, activation="selu")(x2)
x2 = keras.layers.Dense(128, activation="selu")(x2)
x2 = keras.layers.Dense(64, activation="selu")(x2)
x2 = keras.layers.Dropout(0.1)(x2)

# Concatenate both branches
concat = keras.layers.Concatenate()([x1, x2])
```

Model is improving, but nothing too significant over 12 epochs. **Consider increasing the learning rate.**

![[Screenshot_2024-02-20_at_11.23.37_PM.png]]

```Python
Model: "efficient_net_v2b2_backbone"
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━┓
┃ Layer (type)                    ┃ Output Shape              ┃    Param # ┃
┡━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━┩
│ input_layer (InputLayer)        │ (None, None, None, 3)     │          0 │
├─────────────────────────────────┼───────────────────────────┼────────────┤
│ rescaling (Rescaling)           │ (None, None, None, 3)     │          0 │
├─────────────────────────────────┼───────────────────────────┼────────────┤
│ stem_conv (Conv2D)              │ (None, None, None, 32)    │        864 │
├─────────────────────────────────┼───────────────────────────┼────────────┤
│ stem_bn (BatchNormalization)    │ (None, None, None, 32)    │        128 │
├─────────────────────────────────┼───────────────────────────┼────────────┤
│ stem_activation (Activation)    │ (None, None, None, 32)    │          0 │
├─────────────────────────────────┼───────────────────────────┼────────────┤
│ block1a_ (FusedMBConvBlock)     │ (None, None, None, 16)    │      4,672 │
├─────────────────────────────────┼───────────────────────────┼────────────┤
│ block1b_ (FusedMBConvBlock)     │ (None, None, None, 16)    │      2,368 │
├─────────────────────────────────┼───────────────────────────┼────────────┤
│ block2a_ (FusedMBConvBlock)     │ (None, None, None, 32)    │     11,648 │
├─────────────────────────────────┼───────────────────────────┼────────────┤
│ block2b_ (FusedMBConvBlock)     │ (None, None, None, 32)    │     41,600 │
├─────────────────────────────────┼───────────────────────────┼────────────┤
│ block2c_ (FusedMBConvBlock)     │ (None, None, None, 32)    │     41,600 │
├─────────────────────────────────┼───────────────────────────┼────────────┤
│ block3a_ (FusedMBConvBlock)     │ (None, None, None, 56)    │     44,768 │
├─────────────────────────────────┼───────────────────────────┼────────────┤
│ block3b_ (FusedMBConvBlock)     │ (None, None, None, 56)    │    126,560 │
├─────────────────────────────────┼───────────────────────────┼────────────┤
│ block3c_ (FusedMBConvBlock)     │ (None, None, None, 56)    │    126,560 │
├─────────────────────────────────┼───────────────────────────┼────────────┤
│ block4a_ (MBConvBlock)          │ (None, None, None, 104)   │     46,574 │
├─────────────────────────────────┼───────────────────────────┼────────────┤
│ block4b_ (MBConvBlock)          │ (None, None, None, 104)   │    116,090 │
├─────────────────────────────────┼───────────────────────────┼────────────┤
│ block4c_ (MBConvBlock)          │ (None, None, None, 104)   │    116,090 │
├─────────────────────────────────┼───────────────────────────┼────────────┤
│ block4d_ (MBConvBlock)          │ (None, None, None, 104)   │    116,090 │
├─────────────────────────────────┼───────────────────────────┼────────────┤
│ block5a_ (MBConvBlock)          │ (None, None, None, 120)   │    183,962 │
├─────────────────────────────────┼───────────────────────────┼────────────┤
│ block5b_ (MBConvBlock)          │ (None, None, None, 120)   │    229,470 │
├─────────────────────────────────┼───────────────────────────┼────────────┤
│ block5c_ (MBConvBlock)          │ (None, None, None, 120)   │    229,470 │
├─────────────────────────────────┼───────────────────────────┼────────────┤
│ block5d_ (MBConvBlock)          │ (None, None, None, 120)   │    229,470 │
├─────────────────────────────────┼───────────────────────────┼────────────┤
│ block5e_ (MBConvBlock)          │ (None, None, None, 120)   │    229,470 │
├─────────────────────────────────┼───────────────────────────┼────────────┤
│ block5f_ (MBConvBlock)          │ (None, None, None, 120)   │    229,470 │
├─────────────────────────────────┼───────────────────────────┼────────────┤
│ block6a_ (MBConvBlock)          │ (None, None, None, 208)   │    293,182 │
├─────────────────────────────────┼───────────────────────────┼────────────┤
│ block6b_ (MBConvBlock)          │ (None, None, None, 208)   │    672,308 │
├─────────────────────────────────┼───────────────────────────┼────────────┤
│ block6c_ (MBConvBlock)          │ (None, None, None, 208)   │    672,308 │
├─────────────────────────────────┼───────────────────────────┼────────────┤
│ block6d_ (MBConvBlock)          │ (None, None, None, 208)   │    672,308 │
├─────────────────────────────────┼───────────────────────────┼────────────┤
│ block6e_ (MBConvBlock)          │ (None, None, None, 208)   │    672,308 │
├─────────────────────────────────┼───────────────────────────┼────────────┤
│ block6f_ (MBConvBlock)          │ (None, None, None, 208)   │    672,308 │
├─────────────────────────────────┼───────────────────────────┼────────────┤
│ block6g_ (MBConvBlock)          │ (None, None, None, 208)   │    672,308 │
├─────────────────────────────────┼───────────────────────────┼────────────┤
│ block6h_ (MBConvBlock)          │ (None, None, None, 208)   │    672,308 │
├─────────────────────────────────┼───────────────────────────┼────────────┤
│ block6i_ (MBConvBlock)          │ (None, None, None, 208)   │    672,308 │
├─────────────────────────────────┼───────────────────────────┼────────────┤
│ block6j_ (MBConvBlock)          │ (None, None, None, 208)   │    672,308 │
├─────────────────────────────────┼───────────────────────────┼────────────┤
│ top_conv (Conv2D)               │ (None, None, None, 1408)  │    292,864 │
├─────────────────────────────────┼───────────────────────────┼────────────┤
│ top_bn (BatchNormalization)     │ (None, None, None, 1408)  │      5,632 │
├─────────────────────────────────┼───────────────────────────┼────────────┤
│ top_activation (Activation)     │ (None, None, None, 1408)  │          0 │
└─────────────────────────────────┴───────────────────────────┴────────────┘
 Total params: 8,769,374 (33.45 MB)
 Trainable params: 8,687,086 (33.14 MB)
 Non-trainable params: 82,288 (321.44 KB)
```

```Python
Model: "functional_3"
┏━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━┓
┃ Layer (type)        ┃ Output Shape      ┃ Param # ┃ Connected to         ┃
┡━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━┩
│ images (InputLayer) │ (None, 224, 224,  │       0 │ -                    │
│                     │ 3)                │         │                      │
├─────────────────────┼───────────────────┼─────────┼──────────────────────┤
│ efficient_net_v2b2… │ (None, 7, 7,      │ 8,769,… │ images[0][0]         │
│ (EfficientNetV2Bac… │ 1408)             │         │                      │
├─────────────────────┼───────────────────┼─────────┼──────────────────────┤
│ batch_normalizatio… │ (None, 7, 7,      │   5,632 │ efficient_net_v2b2_… │
│ (BatchNormalizatio… │ 1408)             │         │                      │
├─────────────────────┼───────────────────┼─────────┼──────────────────────┤
│ separable_conv2d_2  │ (None, 7, 7,      │ 1,995,… │ batch_normalization… │
│ (SeparableConv2D)   │ 1408)             │         │                      │
├─────────────────────┼───────────────────┼─────────┼──────────────────────┤
│ batch_normalizatio… │ (None, 7, 7,      │   5,632 │ separable_conv2d_2[… │
│ (BatchNormalizatio… │ 1408)             │         │                      │
├─────────────────────┼───────────────────┼─────────┼──────────────────────┤
│ activation_2        │ (None, 7, 7,      │       0 │ batch_normalization… │
│ (Activation)        │ 1408)             │         │                      │
├─────────────────────┼───────────────────┼─────────┼──────────────────────┤
│ features            │ (None, 163)       │       0 │ -                    │
│ (InputLayer)        │                   │         │                      │
├─────────────────────┼───────────────────┼─────────┼──────────────────────┤
│ separable_conv2d_3  │ (None, 7, 7,      │ 1,995,… │ activation_2[0][0]   │
│ (SeparableConv2D)   │ 1408)             │         │                      │
├─────────────────────┼───────────────────┼─────────┼──────────────────────┤
│ dense_4 (Dense)     │ (None, 64)        │  10,496 │ features[0][0]       │
├─────────────────────┼───────────────────┼─────────┼──────────────────────┤
│ global_average_poo… │ (None, 1408)      │       0 │ separable_conv2d_3[… │
│ (GlobalAveragePool… │                   │         │                      │
├─────────────────────┼───────────────────┼─────────┼──────────────────────┤
│ dense_5 (Dense)     │ (None, 32)        │   2,080 │ dense_4[0][0]        │
├─────────────────────┼───────────────────┼─────────┼──────────────────────┤
│ activation_3        │ (None, 1408)      │       0 │ global_average_pool… │
│ (Activation)        │                   │         │                      │
├─────────────────────┼───────────────────┼─────────┼──────────────────────┤
│ dense_6 (Dense)     │ (None, 16)        │     528 │ dense_5[0][0]        │
├─────────────────────┼───────────────────┼─────────┼──────────────────────┤
│ dropout_2 (Dropout) │ (None, 1408)      │       0 │ activation_3[0][0]   │
├─────────────────────┼───────────────────┼─────────┼──────────────────────┤
│ dropout_3 (Dropout) │ (None, 16)        │       0 │ dense_6[0][0]        │
├─────────────────────┼───────────────────┼─────────┼──────────────────────┤
│ concatenate_1       │ (None, 1424)      │       0 │ dropout_2[0][0],     │
│ (Concatenate)       │                   │         │ dropout_3[0][0]      │
├─────────────────────┼───────────────────┼─────────┼──────────────────────┤
│ aux_head (Dense)    │ (None, 6)         │   8,550 │ concatenate_1[0][0]  │
├─────────────────────┼───────────────────┼─────────┼──────────────────────┤
│ head (Dense)        │ (None, 6)         │   8,550 │ concatenate_1[0][0]  │
└─────────────────────┴───────────────────┴─────────┴──────────────────────┘
 Total params: 12,801,114 (48.83 MB)
 Trainable params: 4,321,788 (16.49 MB)
 Non-trainable params: 8,479,326 (32.35 MB)
```