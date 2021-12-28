# GreenScreenCut
绿幕去除
#include <opencv2/core.hpp>
#include <opencv2/imgproc.hpp>
#include <opencv2/highgui.hpp>


cv::Mat element3(3, 3, CV_8U, cv::Scalar(1));
cv::Mat element5(5, 5, CV_8U, cv::Scalar(1));

cv::Mat cutGreenScreen(cv::Mat& src)
{
	cv::Mat srcCut, srcHSV, srcThreshold;
	srcCut = src;

	cvtColor(srcCut, srcHSV, CV_BGR2HSV_FULL);
	cvtColor(srcCut, srcThreshold, CV_BGR2GRAY);

	for (int i = 0; i < srcHSV.rows; i++)
	{
		cv::Vec3b* HSVpixel = srcHSV.ptr<cv::Vec3b>(i);
		uchar* GrayPixel = srcThreshold.ptr<uchar>(i);
		for (int j = 0; j < srcHSV.cols; j++)
		{
			if (HSVpixel[j][0] > 45 && HSVpixel[j][0] < 137 && HSVpixel[j][1] > 43 && HSVpixel[j][2] > 50)
			{
				GrayPixel[j] = 0;
			}
			else
			{
				GrayPixel[j] = 255;
			}
		}
	}	//裁剪绿幕

	dilate(srcThreshold, srcThreshold, element5);
	erode(srcThreshold, srcThreshold, element5);


	//RemoveSmallRegion(videoFrameThreshold, videoFrameThreshold, 20, 1, 0);


	blur(srcThreshold, srcThreshold, cv::Size(7, 7));
	threshold(srcThreshold, srcThreshold, 128, 255, cv::THRESH_BINARY);

	return srcThreshold;
}




int main()
{
	VideoCapture video = VideoCapture("D:\\QQFiles\\459533652\\FileRecv\\01.mp4");
	Mat stream;
	double rate = video.get(CV_CAP_PROP_FPS);
	/*获取视频帧的尺寸*/
	int width = video.get(CV_CAP_PROP_FRAME_WIDTH);
	int height = video.get(CV_CAP_PROP_FRAME_HEIGHT);
	cv::VideoWriter w_cap("D:\\re_video2.avi", CV_FOURCC('M', 'J', 'P', 'G'), rate, cv::Size(width, height));
	while(video.read(stream))
	{
		clock_t start = clock();
		Mat mask = cutGreenScreen(stream);
		Mat result;
		stream.copyTo(result, mask);
		clock_t finish = clock();
		cout << double(finish - start) / (CLOCKS_PER_SEC * 2.5) << endl;
		imshow("result", result);
		waitKey(10);
		w_cap.write(result);
	}
	return 0;
}
