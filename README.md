# Shared_projects

"""
Exercise VII: For this exercise you are asked to implement a function (create_dataset) that returns a tf.data.Dataset
object holding a dataset for image classification. This dataset should read the image and its label from a CSV file
and apply some data augmentation to the images.
At the end of the script you can find some code so that you can run your pipeline and check how fast is it.
To run: `python exercise_7.py {dataset_path} {images_dir}
It will print in your terminal the time cost of generating each batch. Feel free to change to accommodate your needs.
"""
import argparse
import time
import os
import csv

import numpy as np
import tensorflow as tf

        
def create_dataset(dataset_path, images_dir, num_epochs, batch_size):
    """
    :param str dataset_path: The path to the CSV file describing the dataset. Each row should be (image_name, label)
    :param str images_dir: The directory holding the images. The full image path is then the concatenation of the
        images_dir and the image_name obtained from the CSV
    :param int num_epochs: Number of epochs to generate samples
    :param int batch_size: Number of samples per batch
    :return:
    """
    graph = tf.Graph()
    with graph.device("/device:CPU:0"):
        
        generator = data_generator(dataset_path, images_dir)
        dataset = tf.data.Dataset.from_generator(generator, (tf.string, tf.string), output_shape = (tf.TensorShape([]), tf.TensorShape([])))
        
    

    '''
    image_generator = tf.keras.preprocessing.image.ImageDataGenerator(rescale=1./255, )

    train_data_gen = image_generator.flow_from_directory(directory=str(images_dir),
                                                     batch_size=batch_size,
                                                     shuffle=True,
                                                     target_size=(IMG_HEIGHT, IMG_WIDTH),
                                                     classes = list(CLASS_NAMES))

    
    '''
    pass

def data_generator(dataset_path, images_dir):
    with open(dataset_path) as f:
        reader = csv.reader(f, delimiter = ',')
        for image_name, label in reader:
            image_path = os.join(images_dir, image_name)
            yield image_path, label
            
if __name__ == '__main__':
    parser = argparse.ArgumentParser(description='Pipeline execution')
    parser.add_argument('dataset_path', help='Path to dataset description')
    parser.add_argument('images_dir', help='Image directory')
    parser.add_argument('-e', '--num_epochs', type=int, default=1, help='Number of epochs')
    parser.add_argument('-b', '--batch_size', type=int, default=5, help='Batch size')
    args = parser.parse_args()

    with tf.device('/cpu:0'):  # To force the graph operations of the input pipeline to be placed in the CPU
        with tf.name_scope('input_pipeline'):
            dataset = create_dataset(args.dataset_path, args.images_dir, args.num_epochs, args.batch_size)
            iterator = dataset.make_one_shot_iterator()
            batch = iterator.get_next()

    with tf.Session() as sess:
        try:
            while True:
                start = time.time()
                images, labels = sess.run(batch)
                duration = time.time() - start
                print('Time per batch: {}'.format(duration))
        except tf.errors.OutOfRangeError:
            pass
