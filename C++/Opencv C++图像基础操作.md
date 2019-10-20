## 图像基础操作

#### 图像的读取、保存、颜色空间的转换、图像对象的创建

```cpp
#include <iostream>
#include <opencv2/opencv.hpp>

using namespace std;
using namespace cv;

int main()
{
	// 按照指定格式读取图像
	Mat src = imread("F:/OpenCV/img/lenna.jpg");

	// 判断图像是否为空，也就是判断输入图像的路径是否有效
	if (src.empty())
	{
		cout << "could not load a image..." << endl;
		return -1;
	}

	namedWindow("input", WINDOW_AUTOSIZE); // 设置显示图像的窗口性质
	imshow("input", src);

	Mat gray;
	cvtColor(src, gray, COLOR_BGR2GRAY);  // 图像颜色空间的装换，无返回值，通过引用传回转换后的图像
	namedWindow("gray", WINDOW_AUTOSIZE);
	imshow("gray", gray);
	imwrite("F:/OpenCV/img/lenna_gray.jpg", gray);    // 图像保存

	Mat hsv;
	cvtColor(src, hsv, COLOR_BGR2HSV);
	namedWindow("hsv", WINDOW_AUTOSIZE);
	imshow("hsv", hsv);

	waitKey(0);
	return 0;
}
```

**知识点：**

> imread()  按照指定格式读取图像
>
> imwrite()  将图像保存到指定的路径
>
> nameWindow() 设置显示的窗口
>
> cvtColor() 颜色空间转换函数，无返回值，通过传入引用来改变目标图像

#### 图像的赋值、复制与克隆

```cpp
#include <iostream>
#include <opencv2/opencv.hpp>

using namespace std;
using namespace cv;

int main()
{
	// 按照指定格式读取图像
	Mat src = imread("F:/OpenCV/img/lenna.jpg");

	// 判断图像是否为空，也就是判断输入图像的路径是否有效
	if (src.empty())
	{
		cout << "could not load a image..." << endl;
		return -1;
	}

	namedWindow("input", WINDOW_AUTOSIZE); // 设置显示图像的窗口性质
	imshow("input", src);

	// Mat对象的clone，与copy都会产生一个新的存储空间去保存Mat
	Mat m1 = src.clone();

	Mat m2;
	src.copyTo(m2);

	// 图像的赋值，与原来的Mat对象共享数据区，也就是其中一个像素值发生了变化另一个也会发生变化
	Mat m3 = src;

	// 创建空白图像，前面两个比较常用
	Mat m4 = Mat::zeros(src.size(), src.type());  // 创建一个与src同样大小以及同样数据类型的全部为0的空白图像
	Mat m5 = Mat::zeros(Size(512, 512), CV_8UC3); 
	Mat m6 = Mat::ones(Size(51, 512), CV_8UC3);

	// 自定义卷积核
	Mat kernel = (Mat_<char>(3, 3) << 0, -1, 0, -1, 5, -1, 0, -1, 0);

	waitKey();
	return 0;
}
```

**知识点：** 

> src.clone()  图像的克隆, 返回克隆之后的Mat对象
>
> src.copyTo(dst)  无返回值，直接通过引用回传
>
> Mat::zeros() Mat::ones() 创建空白对象
>
> Size() 可以直接定义图像的大小

#### 图像像素的遍历

```cpp
#include <iostream>
#include <opencv2/opencv.hpp>

using namespace std;
using namespace cv;

int main()
{
	// 按照指定格式读取图像
	Mat src = imread("F:/OpenCV/img/lenna.jpg");

	// 判断图像是否为空，也就是判断输入图像的路径是否有效
	if (src.empty())
	{
		cout << "could not load a image..." << endl;
		return -1;
	}

	namedWindow("input", WINDOW_AUTOSIZE); // 设置显示图像的窗口性质
	imshow("input", src);

	int height, width, ch;
	height = src.rows, width = src.cols, ch = src.channels(); // 读取图像的高，宽以及通道数
	cout << height << " " << width << " " << ch << endl;

	// 使用数组进行访问的速度比较慢，因为这里需要遍历数组中的每一个元素
	for (int row = 0; row < height; row++)
	{
		for (int col = 0; col < width; col++)
		{
			if (ch == 3)
			{
				Vec3b bgr = src.at<Vec3b>(row, col); // Vec3b是三个字节的结构体，相当于是一个向量
				bgr[0] = 255 - bgr[0];
				bgr[1] = 255 - bgr[1];
				bgr[2] = 255 - bgr[2];
				src.at<Vec3b>(row, col) = bgr;
			}
			else
			{
				int gray = src.at<uchar>(row, col);
				gray = 255 - gray;
				src.at<uchar>(row, col);
			}
		}
	}
	imshow("result1",src);

	// 使用指针访问，速度快
	Mat result = Mat::zeros(src.size(), src.type());
	int b = 0, g = 0, r = 0;
	int gray = 0;

	for (int row = 0; row < height; row++)
	{
		uchar* curr_row = src.ptr<uchar>(row);
		uchar* result_row = result.ptr<uchar>(row);
		for (int col = 0; col < width; col++)
		{
			if (ch == 3)
			{
				b = *curr_row++;
				g = *curr_row++;
				r = *curr_row++;

				*result_row++ = b;
				*result_row++ = g;
				*result_row++ = r;
			}
			else if (ch == 1)
			{
				gray = *curr_row;
				*result_row = 255 - gray;
			}
		}
	}
	// 因为前面已经对src进行取反了，所以后面不取反与取反之后显示的一样。相当于对取反之后的src图像进行拷贝和克隆
	imshow("result2", result);

	waitKey();
	return 0;
}
```

**知识点：**

> 对图像遍历的两种方式：
>
> 数组访问：注意使用Vec3b数据结构，访问速度慢
>
> 指针访问：使用src.ptr<uchar> (row) 获取当行的首地址，访问速度快

#### 像素的算术操作

```cpp
#include <iostream>

#include <opencv.hpp>
using namespace std;
using namespace cv;

int main()
{
	// 随便读取两张图像
	Mat src1 = imread("F:/OpenCV/data/apple.jpg");
	Mat src2 = imread("F:/OpenCV/data/baboon.jpg");

	// 判断图像是否为空，也就是判断输入图像的路径是否有效
	if (src1.empty() || src2.empty())
	{
		cout << "could not load a image..." << endl;
		return -1;
	}

	imshow("input1", src1);
	imshow("input2", src2);

	int height = src1.rows, width = src1.cols;
	
	// 两张图像进行add
	Mat add_result = Mat::zeros(src1.size(), src1.type());
	add(src1, src2, add_result);
	imshow("add", add_result);

	// 两张图像进行substract
	Mat sub_result = Mat::zeros(src1.size(), src1.type());
	subtract(src1, src2, sub_result);
	imshow("sub", sub_result);

	// 两张图像进行multiply
	Mat mul_result = Mat::zeros(src1.size(), src1.type());
	multiply(src1, src2, mul_result);
	imshow("mul", mul_result);

	// 两张图像进行divide
	Mat div_result = Mat::zeros(src1.size(), src1.type());
	divide(src1, src2, div_result);
	imshow("div", div_result);

	waitKey();
	return 0;
}
```

**知识点：**

> add(src1, src2,dst)
>
> subtract(src1, src2,dst)
>
> multiply(src1, src2,dst)
>
> divide(src1, src2,dst)
>
> 两张图像的乘法与除法，在实际应用中使用的比较少，加法和减法用的多一点。

#### 查找表(Look up table)LUT

```cpp
#include <iostream>
#include <opencv.hpp>

using namespace std;
using namespace cv;

// 自己实现一个LUT查找表，将灰度图像进行二值化
void customColorMap(Mat& img)
{
	int lut[256];
	for (int i = 0; i < 256; i++)
		if (i < 127) lut[i] = 0;
		else lut[i] = 255;

	int h = img.rows, w = img.cols;
	for (int row = 0; row < h; row++)
		for (int col = 0; col < w; col++)
		{
			int pv = img.at<uchar>(row, col);
			img.at<uchar>(row, col) = lut[pv];
		}

	imshow("lut demo", img);
}

int main()
{
	// 随便读取两张图像
	Mat src = imread("F:/OpenCV/data/lena.jpg");

	// 判断图像是否为空，也就是判断输入图像的路径是否有效
	if (src.empty())
	{
		cout << "could not load a image..." << endl;
		return -1;
	}

	imshow("input", src);

	Mat img;
	cvtColor(src, img, COLOR_BGR2GRAY);
	customColorMap(img);

	// 使用查找表函数
	Mat result_hsv;
	applyColorMap(src, result_hsv, COLORMAP_HSV); 
	imshow("result_hsv", result_hsv);

	Mat result_aut;
	applyColorMap(src, result_aut, COLORMAP_AUTUMN);
	imshow("result_aut", result_aut);

	Mat result_win;
	applyColorMap(src, result_win, COLORMAP_WINTER);
	imshow("result_win", result_win);

	Mat result_hot;
	applyColorMap(src, result_hot, COLORMAP_HOT);
	imshow("result_hot", result_hot);

	waitKey();
	return 0;
}


```

**知识点：**

> 查找表就是将图像原来的像素值按照一定的规则，重新映射到另外一些值，来达到不同的要求。查找表的计算效率非常高，只需要进行读取不需要进行计算。
>
> 可以用于制作滤镜，直方图拉伸等等。

