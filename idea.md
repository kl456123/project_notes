



# Frustum PointNet
## disadvantages
* recall is low due to dependence on 2d detector
* may be some redundancy computation(overplap of 2d bbox)

## feature
* instance segmentation




# AVOD
## disadvantages
* computation is time-consuming due to too much anchors
* filter algorithm is of low efficiency
* too bad for small objects
* lack some semantic information

## feature
* RPN


# Idea
1. 2d region proposals in both image and bev map
get 2d region , class info,3d box size
get 2d region , 3d box size
2. do union of both proposals
get initial 3d box(pos and size)
3. do regression for 3d box



