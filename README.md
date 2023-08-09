# Face generation with AI
This simple app generates random images of human faces.

GAN (Generative Adversarial Network) trained on CelebA dataset

Frontend is written in JavaScript. I uses Remix and React.
Backend is written in Python. It uses Tensorflow, Keras.

# Live demo: 
https://facegan.fly.dev

![fce_gn_0](https://github.com/zbigniew54/FaceGAN/assets/132487185/961107e9-5cf6-485e-aa2d-d49228de10be)

You can use sliders to edit character features. Press 'Randomize' to create new character with new set of genes (new latent vector). You can save and mix characters. Note that new (mixed) characters are not created with average of features and genes. Instead they take random value either from first or second source character.

# Improvements
GAN obiously needs more training to remove artifacts and improve image quality. Unfortunately I am currently limited to my laptop without GPU acceleration so this project progres is slow.

# GAN Architecture
GANs consist of two convolutional neural networks: generator and discriminator. Generator is the one that creates image from initial input data. That is 'latent vector' 16 random floats (in range from 0 to 1) and 'control vector' 12 floats (0:1) that define characters features. Discriminator is only used for training the generator. In vanilla GAN it only classifies images created by generator as 'fake' or 'real' but in this implementation it also classifies features of created faces. Each feature is rated separately and discriminator outputs vector of 12 values (one for each feature) that is compared with initial control vector. In this way it encourages generator to create desired image. This is mostly inspired by Auxiliary Classifier GAN (with multi-label classification).

# GAN Training
It is trained in progressive way similarly to StyleGAN. I start from smallest resolution of 14x14, train until it generates reasonable images. That I freeze all layers (in generator and discriminator) and add new layers to handle higher resolution (28x28, 56x56, 112x112). After expansion I only train new layers until model converges. Then I unfreeze old layers (both in genrator and discriminator) and finetune whole model. When it generates nice images I expand model again.
Hyperparameters
* loss function: binary_crossentropy
* learning rate: 4e-6 (for new layers I use ~50e-6 then lower it for fine tuning)
* optimizer: Adam with beta_1=0.0
* batch size: 20  (for new layers I use 32 then lower it to 20 for fine tuning)
* training samples: 2500 images (they are changed every 10 epochs)
* discriminator is trained 3 times per one generator update (this very important for training stability), also discriminator learning rate is x4 time larger

# My GAN training conclusions:
* Most important thing for this model stability is 'label smoothing'. It is so simple yet very effective way for protecting GAN against mode collapse. This is simple technique: instead of comparing discriminator output with 0 (for fake images) or 1 for real ones we 'smooth' those values providing random numbers from 0.0 to 0.1 (for fake images) and 0.8 to 1.0 (for real ones). It is said that only 'real' labels should be smoothed and 'fake' should always be 0 (because smoothing it may encourage generator too keep current behaviour). However I found 'fake' labels smoothing useful in early stage of training because it prevents  discriminator loss (for fake images) from reaching 0.0. I disable 'fake' label smoothing in fine tuning. I also smooth labels for control vector (character features) in similar way.
* Batch normalization is very useful and it helps to converge very fast. Model converges ~5 times faster that without BN. However you have to be very careful with it since it works diffrently in inference and training time. I only use BN in generator.
* ...

# Generator architecture
Note that this show architecture for 12x12 resolution. Training starts from 14x14 so only layers with names startig with 'd0_' are present. Then 'd1_" is added an so on.
![gen_model_d4](https://github.com/zbigniew54/FaceGAN/assets/132487185/6aaf8d86-0772-4b3d-8e43-e9a05bce9a47)

# Discriminator architecture
![discr_model_d4](https://github.com/zbigniew54/FaceGAN/assets/132487185/82d811d4-1d88-4f7f-8510-ebc31a733ff2)


