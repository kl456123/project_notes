

# evaluation code

## getThtresholds
* N_SAMPLE_PTS: 41,max possible sample points
* n_groundtruth: to calculate all possible recall value points
select points from N_SAMPLE_PTS to approximate all possible points

## cleanData

* gt
1. valid class
valid_class = 1 current_class is gt.type
valid_class = 0 current_class is neighboring class
valid_class = -1 current_class is not used

2. ignore
ignore = true if occlusion or truncation or min_height
ignore = false else

ignore_gt.append(0) valid_class ==1 and not ignore 
ignore_gt.append(1) valid_class ==0 or (ignore and valid_class==1)
ignore_gt.append(-1) else

3. dont care

* detection

```
valid_class = 1 det type is current_class
valid_class = -1 else

ignored_det.append(1) height < min_height
ignored_det.append(0) elif valid_class ==1
ignored_det.append(-1) else
```


## computeStatistics
ignored_gt[i] == -1
continue

* fn
valid_detection==NO_DETECTION && ignored_gt[i] ==0
* fp
ignored_det[i]==0
excluding dont care

* tp
valid_detection!==NO_DETECTION
