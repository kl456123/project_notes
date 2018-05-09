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




# gen_mini_batches
class DatasetBuilder
    static function: load_dataset_from_config
    member function: build_kitti_dataset(return KittiDataset)
    copy_config: deep copy

class KittiDataset
    member variable: num_clusters,num_classes,cluster_split,kitti_utils
    member function: _set_up_classes_name
    data_split and data_split_dir
    load_sample_names
    _set_up_directories

class Sample
    __init__(name,augs)


class KittiUtils
includes label clusters utils,mini batch utils(RPN mb,AVOD mb)

## preprocess_input
filter empty anchor and filter diffcult gt, get anchor info(iou and idx)

anchors_info(N x 9):
anchor_indices,ious,offsets,class_idx


# train
config_builder_utils(func)
    model_config:
        path_config:
        1. log dir
        2. pred dir
        3. checkpoint dir
    (copy config file to experiment dir)

## AvodModel(model_config)
### Input config
bev_pixel_size [dims_h,dims_w,bev_depth]
img_pixel_size [dims_h,dims_w,img_depth]

### avod_config
roi_crop_size [7,7]
nms_size: 100
nmns_iou_threshold: 0.01
box_representation: 'box_4ca'
positive_selection: 'not_bkg'

### build(after build rpn model)
get top anchors from rpn model
1. Avod Projection to img and bev space
if expand_xz, x+=expand_length
reorder projected boxes into [y1,x1,y2,x2]

2. Path drop like rpn

3. ROI Pooling  on BEV and image
get bev_rois and img_rois

4. Avod Layers
full connected layers
input rois: bev and img
input_weights: bev_mask and img_mask

fusion_type(early and late,deep)
fusion_method(concat,mean,max)

output by fc(without activation):offset(4 corner and two heights,10=4*2+2) ,cls and orientation(2)

5. Sample mini batch(sample_avod_mini_batch)
like rpn model

6. Decode anchors(convert proposals and gts according to box_rep)

7. ROI summary images in bev and img(in mini batches)

8. apply offset to regress proposals(top anchors from rpn model)

9. Collect tensors into prediction_dict
mini batch predictions
    mb_classifications_logits,mb_classifications_softmax,mb_offsets
mini batch ground truth
    mb_classifications_gt,mb_offsets_gt
top nms predictions
    top_classification_softmax,top_classification_logits,top_prediction_anchors
    top_orientations,top_prediction_boxes_4c,mb_angle_vectors,top_prediction_boxes_3d


### feed
```
firstly get bev_input img_input and label anchors(and label_boxes) from samples
```
1. load_samples:
    sample_dict:label_boxes_3d,label_anchors,label_classes,image_input,bev_input,
    anchors_info(some anchors),point_cloud,ground_plane,calib_p2,name,augs

create_bev_map:height_map(5) and density_map(1)

anchors_iou,anchor_offsets,anchor_classes,anchors_to_use
bev_anchors,img_anchors and something else from sample_dicts

2. fill_anchor_pl_inputs
anchors_ious,anchor_offsets and anchor_classes
bev_anchor and its norm,img_anchor and its norm

3. fill last info
bev_input,image_input
label (label_anchors,label_boxes_3d,label_classes)
calib(ground plane and p2)




## RPNModel

### rpn_config
rpn_proposal_roi_crop_size: 7
fusion_method: "mean"

if val or test,disable drop path

### bev feature extractor

### img feature extractor

### build
1. set up input placeholders
* bev_feature_extractor.preprocess_input(bev_input_batches)(just resize)
* pl_labels
LABEL_ANCHORS,LABEL_BOXES_3D,LABEL_CLASSES
* pl_anchors
  * ANCHORS,ANCHORS_IOUS,ANCHORS_OFFSETS,ANCHORS_CLASSES
  * BEV_ANCHORS_NORM,BEV_ANCHORS
  * IMG_ANCHORS_NORM,IMG_ANCHORS
  * sample_info(calib matrix ,img_idx and ground plane)

2. set up feature extractors
* vgg net for img and bev feature extractor,get feature map
* add bottleneck for both of them

3. create_path_drop_masks(get final_img_mask and final_bev_mask)

4. proposal_roi_pooling and proposal_roi_fusion(mean or concat)
use self._bev_anchors_norm_pl and self._img_anchors_norm_pl to crop inputs

5. anchor_predictor
get objectness(2) and offsets(6), histogram summaries from end points

6. nms
get top_anchors and top_objectness_softmax from doing nms in bev space

7. mini batch sample

8. ROI summary images(bev and img anchors after subsample)

9. GroundTruth Tensor
get objectness_gt from all_iou_gt(1 if greater min_pos_iou else 0)

10. Mask gt and prediction

11. Collect tensors for evaluation
objectness_masked, offsets_masked, offsets_gt_masked,objectness_gt_masked
top_indices,top_anchors,top_objectness_softmax(for prediction)
3d box regression (t_x,t_y,t_z,d_x,d_y,d_z)


### member
1. placeholders
2. placeholder_inputs
3. sample_info



three key
dataset
model
train

## some utils
1. add_placeholder(dtype,shape,name)
2. slim.arg_scope([op1,op2],
                fn=..) (set default parameters for each ops)

3. crop_and_resize
args: images(N,H,W,C)
      boxes(num_boxes,4)
      box_ind(num_boxes,specifies the image the i-th box refers to)
      crop_size
4. anchor_to_box_3d(anchor,fix_lw)
convert anchor format([x,y,z,dimx,dimy,dimz]) to 3d box format([x,y,z,l,w,h,ry])
ry = -pi/2 if rotation


Note that shape of tensor (N,H,W,C)

mini batch label: all_iou_gt,all_offsets_gt and all_classes_gt
