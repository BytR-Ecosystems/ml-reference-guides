# 🖼️ ImageDataGenerator Cheat Sheet

`tensorflow.keras.preprocessing.image.ImageDataGenerator`

*AuDHD-friendly: emoji anchors, no walls of text, clear decision paths*

---

## 🧠 The Big Picture

> **Core concept:** ImageDataGenerator creates modified copies of your training images **on the fly** during training.
>
> - It does **NOT** create new files on disk
> - Each epoch sees different random variations
> - More variation = harder to memorize = better generalization
> - **Only augment TRAINING data. Never augment test/validation data** (except rescale).

> 💡 **Why you need it:**
> - **Small dataset?** → Augmentation simulates having more data
> - **Overfitting?** → Acts as regularization (like Dropout for images)
> - **Model too confident?** → Variations force it to learn real features, not shortcuts

---

## ✂️ Copy-Paste Starter

```python
from tensorflow.keras.preprocessing.image import ImageDataGenerator

# TRAINING: augment + normalize
train_datagen = ImageDataGenerator(
    rescale=1./255,
    rotation_range=20,
    width_shift_range=0.15,
    height_shift_range=0.15,
    shear_range=0.1,
    zoom_range=0.2,
    horizontal_flip=True,
    fill_mode='nearest'
)

# VALIDATION/TEST: normalize ONLY, no augmentation
test_datagen = ImageDataGenerator(rescale=1./255)
```

---

## 🔄 Geometric Transforms

These change the **position/orientation** of the image content:

| Parameter | Type | What It Does | Typical Value |
|---|---|---|---|
| `rotation_range` | int (degrees) | Random rotation up to N degrees | 0–40 |
| `width_shift_range` | float (fraction) | Horizontal shift as fraction of width | 0.1–0.2 |
| `height_shift_range` | float (fraction) | Vertical shift as fraction of height | 0.1–0.2 |
| `shear_range` | float (radians) | Shear intensity (slanting) | 0.0–0.2 |
| `zoom_range` | float or [min,max] | Random zoom in/out | 0.1–0.3 |
| `horizontal_flip` | bool | Randomly flip left-right | `True` for most |
| `vertical_flip` | bool | Randomly flip top-bottom | `False` usually |
| `fill_mode` | str | How to fill empty pixels after transform | `'nearest'` |

---

## 🎨 Pixel-Level Transforms

These change **pixel values** without moving content:

| Parameter | Type | What It Does | Typical Value |
|---|---|---|---|
| `rescale` | float | Multiply every pixel value | `1./255` (normalize to 0–1) |
| `brightness_range` | [min, max] | Random brightness shift | `[0.8, 1.2]` |
| `channel_shift_range` | float | Random channel intensity shift | 0–50 |
| `featurewise_center` | bool | Zero-center per feature (needs `.fit()`) | `False` |
| `samplewise_center` | bool | Zero-center per sample | `False` |
| `featurewise_std_normalization` | bool | Divide by std (needs `.fit()`) | `False` |
| `samplewise_std_normalization` | bool | Divide each sample by its std | `False` |
| `zca_whitening` | bool | ZCA whitening (needs `.fit()`) | `False` (rarely used) |

> 💡 `rescale=1./255` is the most important one. Neural networks train much better on 0–1 range than 0–255. **Always use it.**

---

## 🧱 Fill Modes

When rotation/shift moves pixels outside the frame, `fill_mode` decides what fills the gap:

| fill_mode | Behavior |
|---|---|
| `'nearest'` | Fills with nearest pixel value. **Default.** Usually fine. |
| `'reflect'` | Mirror-reflects at boundary. Good for natural images. |
| `'wrap'` | Wraps around (tiles). Rarely useful. |
| `'constant'` | Fills with `cval` (default 0 = black). Creates visible borders. |

---

## 📥 How to Feed Data to the Model

| Method | When to Use |
|---|---|
| `.flow(X, y)` | Numpy arrays in memory. Data fits in RAM. |
| `.flow_from_directory(path)` | Images in folders. Each subfolder = one class. Large datasets. |
| `.flow_from_dataframe(df, dir)` | DataFrame with filenames + labels. Flexible labeling. |

### Using `.flow()` with numpy arrays (Simpsons setup)

```python
train_datagen = ImageDataGenerator(
    rescale=1./255,
    rotation_range=20,
    horizontal_flip=True,
    zoom_range=0.2
)

train_generator = train_datagen.flow(
    X_train, y_train_ohe,
    batch_size=32
)

model.fit(
    train_generator,
    steps_per_epoch=len(X_train) // 32,
    epochs=100,
    validation_data=(X_test / 255., y_test_ohe)
)
```

### Using `.flow_from_directory()` (reads from folders)

```python
train_generator = train_datagen.flow_from_directory(
    './simpsons_dataset/train/',
    target_size=(100, 100),
    batch_size=32,
    class_mode='categorical'
)

val_generator = test_datagen.flow_from_directory(
    './simpsons_dataset/val/',
    target_size=(100, 100),
    batch_size=32,
    class_mode='categorical'
)

model.fit(train_generator, epochs=100,
          validation_data=val_generator)
```

---

## 🛑 Critical Rules

> **NEVER do this:**
> - ❌ Augment validation or test data (only rescale)
> - ❌ Augment AFTER splitting — augmented copies could leak into test set
> - ❌ Use `vertical_flip` for faces, people, text, or anything with a natural "up"
> - ❌ Use extreme rotation (>45°) unless objects can appear at any angle (cells, satellites)
> - ❌ Forget `rescale` on test data if you used it on training data

> ⚠️ **Double normalization trap:** If you already normalized `X_train` with `/255.0` before passing to `.flow()`, do **NOT** also set `rescale=1./255` in the generator. That would normalize twice (divide by 255²).

---

## 🎯 Task-Specific Presets

Ask yourself: *"What variations would the model see in the real world?"*
Only augment in ways that **preserve the meaning** of the image.

| Task | Suggested Augmentations |
|---|---|
| **Face recognition** | `rotation_range=15`, shift=0.1, `horizontal_flip=True`, NO vertical_flip |
| **Medical imaging** | `rotation_range=360`, both flips, `zoom_range=0.1` (careful with labels) |
| **Text/OCR** | `rotation_range=5`, shift=0.05, NO flips, NO shear |
| **Simpsons classifier** | `rotation_range=20`, shift=0.15, `horizontal_flip=True`, zoom=0.2, shear=0.1 |
| **Satellite/aerial** | `rotation_range=360`, both flips (no "up" in aerial views) |

---

## 🔗 With Transfer Learning

When using a pre-trained model (VGG19, ResNet, etc.), use the model's **built-in preprocessor** instead of rescale:

```python
from tensorflow.keras.applications.vgg19 import preprocess_input

train_datagen = ImageDataGenerator(
    preprocessing_function=preprocess_input,  # replaces rescale
    rotation_range=20,
    horizontal_flip=True,
    zoom_range=0.2
)

test_datagen = ImageDataGenerator(
    preprocessing_function=preprocess_input    # same preprocessing, no augmentation
)
```

> 💡 Each pre-trained model expects a specific input format. VGG expects BGR with per-channel centering, ResNet similar.
> Using `preprocess_input` handles this automatically — **never manually rescale when using transfer learning.**

---

## 🔄 Mental Model: How It Works During Training

```
Epoch 1: image_001 → rotated 12°, shifted left 5%, zoomed 1.1x
Epoch 2: image_001 → rotated -8°, flipped horizontally, zoomed 0.95x
Epoch 3: image_001 → rotated 25°, shifted down 3%, no flip
```

Same image, different view every time. The model can never memorize pixel positions.
It **must** learn actual features (Bart's spiky hair, Lisa's necklace) to classify correctly.

---

## 📋 Deprecation Note

> ℹ️ `ImageDataGenerator` works but is considered **legacy in TF 2.9+**.
> The modern replacement is `tf.keras.layers` (RandomFlip, RandomRotation, RandomZoom, etc.) applied as model layers.
> For coursework and prototyping, ImageDataGenerator is still perfectly fine and widely used.

### Modern equivalent (layers-based)

```python
from tensorflow.keras import layers

data_augmentation = tf.keras.Sequential([
    layers.RandomFlip('horizontal'),
    layers.RandomRotation(0.1),
    layers.RandomZoom(0.2),
])

model = Sequential([
    data_augmentation,
    layers.Rescaling(1./255),
    layers.Conv2D(32, (3,3), activation='relu'),
    # ...
])
```
