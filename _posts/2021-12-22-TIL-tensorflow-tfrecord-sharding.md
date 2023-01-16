---
layout: post
title: "TIL - Tensorflow TFRecord sharding"
date: 2021-12-22
---
I'm currently working on an image segmentation problem to [classify cloud 
organisation patterns from satellite data](https://www.kaggle.com/c/understanding_cloud_organization). 
I'm currently building the input data pipeline for model training and wanted to make use of 
Tensorflow's recommended TFRecord format and their data API.

The training data for this problem is around 5,500 jpg images of clouds and a .csv
file of run length encoded segmentations for each image and cloud formation label. There are
four label names: Fish, Flower,  Gravel and Sugar. 

I want to make the data pipeline as simple as possible, so ahead of time I want to combine the 
run length encodings and images together in TFRecord files. That way when it comes to training 
time the pipeline will only have to read one input, which should give a training speed up in addition
to keeping the code nice and simple.

According to the [Tensorflow documentation on the TFRecord format](https://www.tensorflow.org/tutorials/load_data/tfrecord)
data should be sharded across multiple files so that you can parellelize I/O, whilst keeping each file 10MB+ and ideally 100MB+.

The following code shows the implementation of this for my data. I chose 20 shards here, which gives files
around 200MB, I may adjust this as I come to training time, depending on the machine I will train on.
The code takes around 10mins to run for the ~5,500 1400x2100 images so it is relatively quick to iterate
if required.

First the imports and helper functions:
```python
from tqdm import tqdm
import tensorflow as tf
import pandas as pd

LABELS = ["Fish", "Flower", "Gravel", "Sugar"]
# made up folder name
INPUT_PATH = "gs://folder/data/train.csv"

def get_labels(data_path):
    """
    Reads the training data from data_path and outputs a {image_name: [rle]} dict.
    """
    train_df = pd.read_csv(data_path)

    # Rename the columns to correct format
    new_names = ["image_label", "encoded_pixels"]
    rename_dict = dict(zip(train_df.columns, new_names))
    train_df = train_df.rename(columns=rename_dict)

    # Split out the image ID from the label name
    train_df['image_id'] = train_df['image_label'].apply(lambda x: x.split('_')[0].split('.')[0])
    train_df['label'] = train_df['image_label'].apply(lambda x: x.split('_')[1])
    train_df.loc[train_df["encoded_pixels"].isna(), "encoded_pixels"] = ""

    # Group by image_id
    grouped_rles = train_df.groupby(["image_id"]).agg({'encoded_pixels': lambda x: list(x),
                                                       'label': lambda x: list(x)})

    # Ensure the label values are in the correct order
    assert all(element == LABELS for element in grouped_rles['label'].values)

    return dict(zip(grouped_rles.index, grouped_rles['encoded_pixels']))


def _bytes_feature(value):
    """
    Returns a bytes_list from a string / byte.
    """
    if isinstance(value, type(tf.constant(0))):
        value = value.numpy()  # BytesList won't unpack a string from an EagerTensor.
    return tf.train.Feature(bytes_list=tf.train.BytesList(value=[value]))


def _int64_feature(value):
    """
    Returns an int64_list from a bool / enum / int / uint.
    """
    return tf.train.Feature(int64_list=tf.train.Int64List(value=[value]))


def serialize_image(image_string, label):
    """
    Creates a tf.train.Example message ready to be written to a file.
    """
    image_shape = tf.io.decode_jpeg(image_string).shape
    
    # Create a dictionary mapping the feature name to the tf.train.Example-compatible
    # data type.
    feature = {
        'height': _int64_feature(image_shape[0]),
        'width': _int64_feature(image_shape[1]),
        'depth': _int64_feature(image_shape[2]),
        'image_raw': _bytes_feature(image_string),
        'fish': _bytes_feature(label[0].encode('utf-8')),
        'flower': _bytes_feature(label[1].encode('utf-8')),
        'gravel': _bytes_feature(label[2].encode('utf-8')),
        'sugar': _bytes_feature(label[3].encode('utf-8')),
    }

    # Create a Features message using tf.train.Example.

    example_proto = tf.train.Example(features=tf.train.Features(feature=feature))
    return example_proto.SerializeToString()


def chunks(lst, n):
    """Yield successive n-sized chunks from lst."""
    for i in range(0, len(lst), n):
        yield lst[i:i + n]
```

And the implementation:
```python
N_SHARDS = 20

# Get the label dictionary and image key names
labels = get_labels(INPUT_PATH)
img_keys = list(labels.keys())

# Get generator of N_SHARD image key chunks 
img_key_shards = chunks(img_keys, int(len(img_keys) / N_SHARDS))

# Create N_SHARDS TFRecord files
for i, img_keys in tqdm(enumerate(img_key_shards)):
    idx = str(i).zfill(4)
    tf_record_path = f"raw_train_images_{idx}.tfrecords"
    with tf.io.TFRecordWriter(tf_record_path) as writer:
        for img_key in img_keys:
            # made up folder 
            image_string = tf.io.read_file(f'gs://folder/data/train/images/{img_key}.jpg')
            serialized_img = serialize_image(image_string, labels[img_key])
            writer.write(serialized_img)
```
