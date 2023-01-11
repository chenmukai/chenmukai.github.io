---
title: opencv
date: 2022-08-15 09:24:53
tags:
	- opencv,c++
categories:
	- opencv
keywords:"opencv"
---

# 1.读取显示图像

## 1.图片读取

```c++
#include<opencv2/imgcodecs.hpp>
#include<opencv2/highgui.hpp>
#include<opencv2/imgproc.hpp>
#include<iostream>

using namespace cv;
using namespace std;

/// <summary>
/// Images
/// </summary>
void main() {
	string path = "Resources/test.png"; //图片路径
	Mat img = imread(path);
	imshow("Image", img);
	waitKey(0);
}
```

## 2.视频读取

```c++
#include<opencv2/imgcodecs.hpp>
#include<opencv2/highgui.hpp>
#include<opencv2/imgproc.hpp>
#include<iostream>

using namespace cv;
using namespace std;

/// <summary>
/// Videos
/// </summary>
void main() {
	string path = "Resources/test_video.mp4"; //视频路径
	VideoCapture cap(path);
	Mat img;
	while (true) {
		cap.read(img);
		imshow("Image", img);
		waitKey(20);
	}
}
```

## 3.摄像头读取

```c++
#include<opencv2/imgcodecs.hpp>
#include<opencv2/highgui.hpp>
#include<opencv2/imgproc.hpp>
#include<iostream>

using namespace cv;
using namespace std;

/// <summary>
/// Webcam
/// </summary>
void main() {
	VideoCapture cap(0);
	Mat img;
	while (true) {
		cap.read(img);
		imshow("Image", img);
		waitKey(1);
	}
}
```

# 2.图像操作

## 1.图像预处理

```c++
#include<opencv2/imgcodecs.hpp>
#include<opencv2/highgui.hpp>
#include<opencv2/imgproc.hpp>
#include<iostream>

using namespace cv;
using namespace std;

/// <summary>
/// Basic Functions
/// </summary>
void main() {
	string path = "Resources/test.png";
	Mat img = imread(path);

	Mat imgGray,imgBlur,imgCanny,imgDil,imgErode;
	cvtColor(img, imgGray, COLOR_BGR2GRAY); 
	//高斯滤波
	GaussianBlur(img, imgBlur, Size(3, 3), 5, 0);
	//canny边缘检测
	Canny(imgBlur, imgCanny, 50, 150);

	Mat kernel = getStructuringElement(MORPH_RECT, Size(5, 5));
	//膨胀
	dilate(imgCanny, imgDil, kernel);
	//腐蚀
	erode(imgDil, imgErode, kernel);

	imshow("Image", img);
	imshow("Gray Image", imgGray);
	imshow("GaussionBlur Image", imgBlur);
	imshow("Canny Image", imgCanny);
	imshow("Dilate Image", imgDil);
	imshow("Erode Image", imgErode);
	waitKey(0);
}
```

## 2.图像压缩裁剪

```c++
#include<opencv2/imgcodecs.hpp>
#include<opencv2/highgui.hpp>
#include<opencv2/imgproc.hpp>
#include<iostream>

using namespace cv;
using namespace std;

/// <summary>
/// Resize and crop
/// </summary>
void main() {
	string path = "Resources/test.png";
	Mat img = imread(path);

	Mat imgResize,imgCrop;
	//cout << img.size() << endl;
	//resize(img, imgResize, Size(640, 480)); //按自定义宽高进行缩放
	resize(img, imgResize, Size(), 0.5, 0.5); //等比例缩放

	Rect roi(200,100,300,300); 
	imgCrop = img(roi);
	imshow("Image", img);
	imshow("resize img", imgResize);
	imshow("crop img", imgCrop);
	waitKey(0);
}
```

## 3.绘制形状添加文本

```c++
#include<opencv2/imgcodecs.hpp>
#include<opencv2/highgui.hpp>
#include<opencv2/imgproc.hpp>
#include<iostream>

using namespace cv;
using namespace std;

/// <summary>
/// Draw shapes and text
/// </summary>
void main() {
	
	//Blank Image
	Mat img(512, 512, CV_8UC3, Scalar(255, 255, 255));
	circle(img, Point(256, 256), 155, Scalar(0, 69, 255),FILLED);
	rectangle(img, Point(130, 226), Point(382, 286), Scalar(255, 255, 255), FILLED);
	line(img, Point(130, 296), Point(382, 296), Scalar(255, 255, 255), 2);
	putText(img, "Lucas's Workshop", Point(137, 262), FONT_HERSHEY_DUPLEX, 0.75, Scalar(0, 69, 255), 2);

	imshow("Image", img);
	waitKey(0);
}
```

4.透视投影变换矫正

```c++
#include<opencv2/imgcodecs.hpp>
#include<opencv2/highgui.hpp>
#include<opencv2/imgproc.hpp>
#include<iostream>

using namespace cv;
using namespace std;

/// <summary>
/// Warp Images
/// </summary>
 
float w = 250, h = 350;
Mat matrix, imgWarp;
void main() {
	string path = "Resources/cards.jpg";
	Mat img = imread(path);

	Point2f src[4] = { {529,142},{771,190}, {405,395},{674,457} };
	Point2f dst[4] = { {0.0f,0.0f},{w,0.0f}, {0.0f,h},{w,h} };

	matrix = getPerspectiveTransform(src, dst); //获取透视变换矩阵
	warpPerspective(img, imgWarp, matrix, Point(w, h));

	for (int i = 0; i < 4; i++) {
		circle(img, src[i], 10, Scalar(0, 0, 255), FILLED);
	}

	imshow("Image", img);
	imshow("Warp Image", imgWarp);
	waitKey(0);
}
```

# 3.颜色检测

```c++
#include<opencv2/imgcodecs.hpp>
#include<opencv2/highgui.hpp>
#include<opencv2/imgproc.hpp>
#include<iostream>

using namespace cv;
using namespace std;

/// <summary>
/// Color Detection
/// </summary>

Mat imgHSV,mask;
int hmin = 0, smin = 0, vmin = 0;
int hmax = 179, smax = 255, vmax = 255;

void main() {
	string path = "Resources/shapes.png";
	Mat img = imread(path);

	cvtColor(img, imgHSV, COLOR_BGR2HSV);
	namedWindow("Trackbars", (640, 200));
	createTrackbar("Hue Min", "Trackbars", &hmin, 179);
	createTrackbar("Hue Max", "Trackbars", &hmax, 179);
	createTrackbar("Sat Min", "Trackbars", &smin, 255);
	createTrackbar("Sat Max", "Trackbars", &smax, 255);
	createTrackbar("Val Min", "Trackbars", &vmin, 255);
	createTrackbar("Val Max", "Trackbars", &vmax, 255);

	while (true) {
		Scalar lower(hmin, smin, vmin);
		Scalar upper(hmax, smax, vmax);
		inRange(imgHSV, lower, upper, mask);

		imshow("Image", img);
		imshow("HSV Image", imgHSV);
		imshow("Mask Image", mask);
		waitKey(1);
	}
}
```

# 4.形状检测

```c++
#include<opencv2/imgcodecs.hpp>
#include<opencv2/highgui.hpp>
#include<opencv2/imgproc.hpp>
#include<iostream>

using namespace cv;
using namespace std;

/// <summary>
/// Shapes Detection
/// </summary>

Mat imgGray, imgBlur, imgCanny, imgDil, imgErode;

void getContours(Mat imgDil, Mat img) {
	vector<vector<Point>> contours;
	vector<Vec4i> hierarchy;
	findContours(imgDil, contours, hierarchy, RETR_EXTERNAL, CHAIN_APPROX_SIMPLE);
	//drawContours(img, contours, -1, Scalar(0, 0, 0), 2);
	
	vector<vector<Point>> conPoly(contours.size());
	vector<Rect> boundRect(contours.size());
	string objectType;
	for (int i = 0; i < contours.size(); i++) 
	{
		int area = contourArea(contours[i]);
		cout << area << endl;
		if (area > 1000) 
		{
			float peri = arcLength(contours[i],true);
			approxPolyDP(contours[i], conPoly[i], 0.02 * peri, true);
			//drawContours(img, contours, i, Scalar(0, 0, 0), 2);
			
			cout << conPoly[i].size() << endl;
			boundRect[i] = boundingRect(conPoly[i]);
			
			int objCor = (int)conPoly[i].size();
			if (objCor == 3) { objectType = "Tri"; }
			else if (objCor == 4) 
			{ 
				float aspRatio = (float)boundRect[i].width / (float)boundRect[i].height;
				if (aspRatio > 0.95 && aspRatio < 1.05) { objectType = "Square"; }
				else { objectType = "Rect"; }
			}
			else if (objCor > 4) { objectType = "Circle"; }
			
			drawContours(img, conPoly, i, Scalar(0, 0, 0), 2);
			rectangle(img, boundRect[i].tl(), boundRect[i].br(), Scalar(0, 255, 0), 5);
			putText(img, objectType, { boundRect[i].x,boundRect[i].y - 5 }, FONT_HERSHEY_PLAIN, 1, Scalar(0, 69, 255), 1.5);
		}
	}
}

void main() {
	string path = "Resources/shapes.png";
	Mat img = imread(path);

	//Preprocessing
	cvtColor(img, imgGray, COLOR_BGR2GRAY);
	GaussianBlur(imgGray, imgBlur, Size(3, 3), 3, 0);
	Canny(imgBlur, imgCanny, 25, 75);
	Mat kernel = getStructuringElement(MORPH_RECT, Size(3, 3));
	dilate(imgCanny, imgDil, kernel);

	getContours(imgDil, img);

	imshow("Image", img);
	//imshow("Gray Image", imgGray);
	//imshow("Blur Image", imgBlur);
	//imshow("Canny Image", imgCanny);
	//imshow("Dilate Image", imgDil);
	waitKey(0);
}
```

# 5.人脸检测

```c++
#include<opencv2/imgcodecs.hpp>
#include<opencv2/highgui.hpp>
#include<opencv2/imgproc.hpp>
#include<opencv2/objdetect.hpp>
#include<iostream>

using namespace cv;
using namespace std;

/// <summary>
/// Face Detection
/// </summary>
void main() {
	string path = "Resources/mytest.jpg";
	Mat img = imread(path);

	CascadeClassifier faceCascade;
	faceCascade.load("Resources/haarcascade_frontalface_default.xml");

	if (faceCascade.empty()) 
	{
		cout << "xml file not loaded" << endl;
	}

	vector<Rect> faces;
	faceCascade.detectMultiScale(img, faces, 1.1, 10);

	for (int i = 0; i < faces.size(); i++) 
	{
		rectangle(img, faces[i].tl(), faces[i].br(), Scalar(255, 0, 255), 3);
	}
	imshow("Image", img);
	waitKey(0);
}
```

# 6.project

## 1.虚拟画笔作画

```c++
#include<opencv2/imgcodecs.hpp>
#include<opencv2/highgui.hpp>
#include<opencv2/imgproc.hpp>
#include<iostream>

using namespace cv;
using namespace std;

/// <summary>
/// project1 虚拟画笔作画
/// </summary>

Mat img;
vector<vector<int>> newPoints;
//<< hmin,smin,vmin,hmax,smax,vmax >>
vector<vector<int>> myColors{ {124,48,117,143,170,255},{68,72,156,102,126,255} };
vector<Scalar> myColorValues{ {255,0,255},{0,255,0} };

Point getContours(Mat input_img) {
	vector<vector<Point>> contours;
	vector<Vec4i> hierarchy;
	findContours(input_img, contours, hierarchy, RETR_EXTERNAL, CHAIN_APPROX_SIMPLE);
	//drawContours(img, contours, -1, Scalar(0, 0, 0), 2);

	vector<vector<Point>> conPoly(contours.size());
	vector<Rect> boundRect(contours.size());
	Point myPoint(0, 0);
	for (int i = 0; i < contours.size(); i++)
	{
		int area = contourArea(contours[i]);
		cout << area << endl;
		if (area > 1000)
		{
			float peri = arcLength(contours[i], true);
			approxPolyDP(contours[i], conPoly[i], 0.02 * peri, true);
			//drawContours(img, contours, i, Scalar(0, 0, 0), 2);

			cout << conPoly[i].size() << endl;
			boundRect[i] = boundingRect(conPoly[i]);
			myPoint.x = boundRect[i].x + boundRect[i].width / 2;
			myPoint.y = boundRect[i].y;
			
			drawContours(img, conPoly, i, Scalar(0, 0, 0), 2);
			rectangle(img, boundRect[i].tl(), boundRect[i].br(), Scalar(0, 255, 0), 5);

		}
	}
	return myPoint;
}

vector<vector<int>> findColor(Mat img) {
	Mat imgHSV;
	cvtColor(img, imgHSV, COLOR_BGR2HSV);

	for (int i = 0; i < myColors.size(); i++)
	{
		Scalar lower(myColors[i][0], myColors[i][1], myColors[i][2]);
		Scalar upper(myColors[i][3], myColors[i][4], myColors[i][5]);
		Mat mask;
		inRange(imgHSV, lower, upper, mask);
		//imshow(to_string(i), mask);
		Point myPoint = getContours(mask);
		if (myPoint.x != 0 && myPoint.y != 0) {
			newPoints.push_back({ myPoint.x,myPoint.y,i });
		}
	}
	return newPoints;
}

void drawOnCanvas(vector<vector<int>> newPoints, vector<Scalar>myColorValues) {
	for (int i = 0; i < newPoints.size(); i++)
	{
		circle(img, Point(newPoints[i][0],newPoints[i][1]), 10, myColorValues[newPoints[i][2]], FILLED);
	}
}

void main() {

	VideoCapture cap(0);

	while (true) {
		cap.read(img);
		newPoints = findColor(img);
		drawOnCanvas(newPoints, myColorValues);

		imshow("Image", img);
		waitKey(1);
	}
}
```

## 2.文档扫描

```c++
#include<opencv2/imgcodecs.hpp>
#include<opencv2/highgui.hpp>
#include<opencv2/imgproc.hpp>
#include<iostream>

using namespace cv;
using namespace std;

/// <summary>
/// project2 文档扫描
/// </summary>

Mat imgOriginal, imgGray, imgBlur, imgCanny, imgDil, imgErode, imgThre, imgWarp, imgCrop;
vector<Point> initialPoints, docPoints;
float w = 420, h = 596;

Mat preProcessing(Mat img) {
	cvtColor(img, imgGray, COLOR_BGR2GRAY);
	GaussianBlur(imgGray, imgBlur, Size(3, 3), 3, 0);
	Canny(imgBlur, imgCanny, 25, 75);

	Mat kernel = getStructuringElement(MORPH_RECT, Size(3, 3));
	dilate(imgCanny, imgDil, kernel);

	return imgDil;

}

vector<Point> getContours(Mat input_img) {
	vector<vector<Point>> contours;
	vector<Vec4i> hierarchy;
	findContours(input_img, contours, hierarchy, RETR_EXTERNAL, CHAIN_APPROX_SIMPLE);
	//drawContours(img, contours, -1, Scalar(0, 0, 0), 2);

	vector<vector<Point>> conPoly(contours.size());
	vector<Rect> boundRect(contours.size());
	vector<Point> biggest;
	int maxArea = 0;
	for (int i = 0; i < contours.size(); i++)
	{
		int area = contourArea(contours[i]);
		cout << area << endl;
		if (area > 1000)
		{
			float peri = arcLength(contours[i], true);
			approxPolyDP(contours[i], conPoly[i], 0.02 * peri, true);
			//drawContours(img, contours, i, Scalar(0, 0, 0), 2);

			if (area > maxArea && conPoly[i].size()==4)
			{
				//drawContours(imgOriginal, conPoly, i, Scalar(255, 0, 255), 5);
				biggest = { conPoly[i][0],conPoly[i][1] ,conPoly[i][2] ,conPoly[i][3] };
				maxArea = area;
			}
			//rectangle(imgOriginal, boundRect[i].tl(), boundRect[i].br(), Scalar(0, 255, 0), 5);

		}
	}
	return biggest;
}

void drawPoints(vector<Point> points, Scalar color) {
	for (int i = 0; i < points.size(); i++) 
	{
		circle(imgOriginal, points[i], 10, color, FILLED);
		putText(imgOriginal, to_string(i), points[i], FONT_HERSHEY_PLAIN, 2, color, 2);
	}
}


vector<Point> reorder(vector<Point>points) {

	vector<Point> newPoints;
	vector<int> sumPoints, subPoints;
	for (int i = 0; i < 4; i++) {
		sumPoints.push_back(points[i].x + points[i].y);
		subPoints.push_back(points[i].x - points[i].y);
	}
	
	newPoints.push_back(points[min_element(sumPoints.begin(), sumPoints.end()) - sumPoints.begin()]);
	newPoints.push_back(points[max_element(subPoints.begin(), subPoints.end()) - subPoints.begin()]);
	newPoints.push_back(points[min_element(subPoints.begin(), subPoints.end()) - subPoints.begin()]);
	newPoints.push_back(points[max_element(sumPoints.begin(), sumPoints.end()) - sumPoints.begin()]);

	return newPoints;

}

Mat getWarp(Mat img, vector<Point> points, float w, float h) {
	Point2f src[4] = { points[0],points[1],points[2],points[3]};
	Point2f dst[4] = { {0.0f,0.0f},{w,0.0f}, {0.0f,h},{w,h} };

	Mat matrix = getPerspectiveTransform(src, dst);
	warpPerspective(img, imgWarp, matrix, Point(w, h));

	return imgWarp;
}

void main() {
	string path = "Resources/paper.jpg";
	imgOriginal = imread(path);
	//resize(imgOriginal, imgOriginal, Size(), 0.5, 0.5);

	//preprocessing
	imgThre = preProcessing(imgOriginal);

	//Get Contours - Biggest
	initialPoints = getContours(imgThre);
	//drawPoints(initialPoints, Scalar(0, 0, 255));
	docPoints = reorder(initialPoints);
	//drawPoints(docPoints, Scalar(0, 255, 0));
	//Warp
	imgWarp = getWarp(imgOriginal, docPoints, w, h);
	//Crop
	int croVal = 10;
	Rect roi(croVal, croVal, w - (2 * croVal), h - (2 * croVal));
	imgCrop = imgWarp(roi);

	imshow("Image", imgOriginal);
	imshow("propressed img", imgThre);
	imshow("warp img", imgWarp);
	imshow("crop img", imgCrop);
	waitKey(0);
}
```

## 3.车牌定位

```c++
#include<opencv2/imgcodecs.hpp>
#include<opencv2/highgui.hpp>
#include<opencv2/imgproc.hpp>
#include<opencv2/objdetect.hpp>
#include<iostream>

using namespace cv;
using namespace std;

/// <summary>
/// project3 车牌定位
/// </summary>

void main() {
	VideoCapture cap(0);
	Mat img;

	CascadeClassifier plateCascade;
	plateCascade.load("Resources/haarcascade_russian_plate_number.xml");

	if (plateCascade.empty())
	{
		cout << "xml file not loaded" << endl;
	}

	vector<Rect> plates;

	while (true) {
		cap.read(img);

		plateCascade.detectMultiScale(img, plates, 1.1, 10);

		for (int i = 0; i < plates.size(); i++)
		{
			Mat imgCrop = img(plates[i]);
			//imshow(to_string(i), imgCrop);
			imwrite("Resources/Plates/" + to_string(i) + ".png",imgCrop);
			rectangle(img, plates[i].tl(), plates[i].br(), Scalar(255, 0, 255), 3);
		}

		imshow("Image", img);
		waitKey(1);
	}
}
```

