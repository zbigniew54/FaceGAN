# Face generation with AI
This simple app generates random images of human faces.

GAN (Generative Adversarial Network) trained on CelebA dataset

Frontend is written in JavaScript. I uses Remix and React.
Backend is written in Python. It uses Tensorflow, Keras.

# Live demo: 
https://facegan.fly.dev
You can use sliders to edit character features. Press 'Randomize' to create new character with new set of genes (new latent vector). You can save and mix characters. Note that new (mixed) characters are not created with average of features and genes. Instead they take random value either from first or second source character.

# Improvements
GAN obiously needs more training to remove artifacts and improve image quality. Unfortunately I am currently limited to my laptop without GPU acceleration so this project progres is slow.

# GAN Architecture
GANs consist of two convolutional neural networks: generator and discriminator. Generator is the one that creates image from initial input data. That is 'latent vector' 16 random floats (in range from 0 to 1) and 'control vector' 12 floats (0:1) that define characters features. Discriminator is only used for training the generator. In vanilla GAN it only classifies images created by generator as 'fake' or 'real' but in this implementation it also classifies features of created faces. Each feature is rated separately and discriminator outputs vector of 12 values (one for each feature) that is compared with initial control vector. In this way it encourages generator to create desired image. This is mostly inspired by Auxiliary Classifier GAN (with multi-label classification).

# GAN Training
It is trained in progressive way similarly to StyleGAN. I start from smallest resolution of 14x14, train until it generates reasonable images. That I freeze all layers (in generator and discriminator) and add new layers to handle higher resolution (28x28, 56x56, 112x112). After expansion I only train new layers until model converges. Then I unfreeze old layers (both in genrator and discriminator) and finetune whole model. When it generates nice images I expand model again.

# My GAN training conclusions:
