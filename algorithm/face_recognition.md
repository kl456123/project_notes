# Face Recognition



## Face Detection
* use image pyramid and sliding window to generate proposals,
extract LAB features and boosted cascade classifiers to filter amounts of
negative no-faces regions.

* use surf feature and coarse mlp cascade

* single mlp cascade with shape-indexed feature


## Face Alignment(Face Landmark Detection)
1. cascade mlp with shape-indexed feature to regression offset relate to
the previous prediction


## Face Identification
1. face detection like before.
2. face landmark detection like before.
3. calculate transform matrix and spatial transform source image.
4. deep neural network to get 125d feature embedding.
4. calc similarity score of source and target.
