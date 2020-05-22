
# rpn
## generate_anchor(two method)
* 
    1. shift + scales_ratios_anchors(in a location)
    2. np.meshgrid

*
    1. grid_anchor_generator
    2. multiple_grid_anchor_generator
    3. multiscale_grid_anchor_generator
* args
    1. base_anchor(equal to feature stride)
    2. scales(dim ratios not square ratios)
    3. ratios
* other args
    1. anchor_strides and anchor_offsets
    place anchor in the image(can be normalized)
    2. min_level and max_level and anchor_scale
    get base_anchor:[2**level* anchor_scale,...]
    3. feat_stride
    4. feat_shape


## some weights
```
Loss = 1/N_cls\sum_{L_cls(p_i,p_i^*)} + \lambda 1/N_reg \sum_{p*L_reg(t_i,t_i^*)}
```
* rpn_inside weight
```
bbox_inside_weights = np.zeros((len(inds_inside), 4), dtype=np.float32)
bbox_inside_weights[labels == 1, :] = np.array(cfg.TRAIN.RPN_BBOX_INSIDE_WEIGHTS)
```
* rpn_outside weight
```
if rpn_positive_weight==-1:
    //uniform sample weights
    negative_weight = positive_weight = 1/num_examples
else:
    positive_weight = rpn_positive_weight/np.sum(label==1)
    negative_weight = (1-rpn_positive_weight)/np.sum(label==1)
```
* rcnn_inside_weight
```
bbox_inside_weights = np.zeros((len(inds_inside), 4*num_classes), dtype=np.float32)
bbox_weights[index, start:end] = cfg.TRAIN.BBOX_WEIGHTS
```
* rcnn_outside_weight
np.array(rcnn_inside_weight>0)

for rpn_box loss,like rpn_box_weight*loss(rpn_box-rpn_box_target)
* bbox_weights_ohem

## pre_nms_topN and post_nms_topN




Note that proposals is (5,N),the first dim is batch_inds




# sample_rois
all_rois include rpn_rois and gts
sample fgs and bgs by using npr.choice()
## universal sample steps:
1. subsample  samples sperate from dont care
2. subsample samples dividing to neg labels and pos labels
3. subsample samples from specific samples set(pos or neg) to get enough


## some layer

* anchor_target_layer
assign anchor to gt targets ,produce anchor classification labels and bounding-box regression targets
something should noted here:
1. rpn_bbox_target,zero for bg and _compute_target(anchor,gt) for fg
2. label,1 is positive,0 is negative ,-1 is don't care
3. diable something(pos or neg) if too many
```
bg = 0
labels[gt_argmax_overlaps] = 1
labels[max_overlaps >= cfg.TRAIN.RPN_POSITIVE_OVERLAP] = 1
// bg 0
```

* proposal_target_layer
assign object detection proposals to gt targets, produces proposal classification labels
and bounding-box regression targets

* proposal_layer
outputs object detection proposals by applying estimated bounding-box transformations to a set of regular
boxes (called "anchors")

* Note that xx_target layer is used for generating target to calculate loss,target needs to be sampled(balanced sample)
as for proposal layer, it needs to be nms to reduce the number
