# Config
1. model_config
2. dataset_config
3. train_config



## trainer
```
trainer.train(model,train_config)
```

## model

* AvodModel
* RpnModel


## dataset
use dataset_config to generate



# some points
1. Generate3dAnchors
area_extents:[[min_x,max_x],[min_y,max_y],[min_z,max_z]]
anchor_3d_sizes: list of 3d anchor sizes N x (l,w,h)
return boxes: list of 3D anchors in box_3d format N x [x,y,z,l,w,h,ry]

calculate y according to x,z due to anchors on the ground plane

2. Build
- input placeholders
bev_input,
img_input,
pl_labels,
pl_anchors:bev_anchor_projections,img_anchor_projections,sample_info(camera calibration,image_idx,groud plane)
- feature extractor
get feature map and end points from bev_feature_extractor
img_feature_extractor is the same
bottleneck(1x1 conv)(get proposal)
- path drop randomly
- proposals RoI pooling
- fusion
- avod projection
project top anchors to bev(normalized corner)
project top anchors to image
RoI pooling(use tf.image.crop_and_resize)
get orentation vector(unit vector)
construct minibatch and subsample in it
