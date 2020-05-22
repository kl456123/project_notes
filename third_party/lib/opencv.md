# some points
1. reference count
2. image type(color,greyscale,..)


* imread(filename,img_type)
* namedWindow(name,WINDOW_AUTOSIZE)
* imshow(string,Mat)

* Rect(x1,y1,x2,y2)
* Range()
* Mat(row_range,col_range)
* Mat(Mat,Rect)

note that they can be used for get ROI from a image

please use Mat.clone() and Mat::copyTo()

don't need to think about memory management with OpenCVs c++ interface


# data type
CV_8UC1   // 8位无符号单通道  
CV_8UC3   // 8位无符号3通道  
CV_8UC4  
CV_32FC1  // 32位浮点型单通道  
CV_32FC3  // 32位浮点型3通道  
CV_32FC4  

# how to create
1. read from image
2. create manual
cv::Mat m = cv::Mat::ones(cv::Size(5,5),data_type)
3. resize(src,dst,Size(),fx,fy,interpolation)

# matrix operation
* m.t()
* m.inv()

countNonZero(m)
meanStdDev(src,mean,stddev)

minMaxLoc()

colRange()
rowRange()