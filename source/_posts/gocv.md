---
title: gocv的使用
date: 2019-08-07 18:50:39
tags:
- opencv
- golang
categories:
- golang
- flask学习笔记
preview: 100
---

# gocv

### 读取图片

```go
	img := gocv.IMRead("test.png", gocv.IMReadColor)
	win := gocv.NewWindow("hi")
	win.IMShow(img)
	win.WaitKey(0)
```

### 创建纯色图片
```go
func CreateImgByBGR(sizex int, sizey int, b float64, g float64, r float64) gocv.Mat {
	img := gocv.NewMatWithSizeFromScalar(gocv.NewScalar(b, g, r, 255), sizex, sizey, gocv.MatTypeCV8UC3)
	return img
}
```
### 转换
```go
	img := gocv.IMRead("test.png", gocv.IMReadColor)
	dst := gocv.NewMat()
	gocv.CvtColor(img, &dst, gocv.ColorBGRToHSV)
	win := gocv.NewWindow("hi")
	win.IMShow(dst)
	win.WaitKey(0)
```

### inRange
```go
func main() {
	img := gocv.IMRead("test.png", gocv.IMReadColor)
	lb := gocv.NewScalar(68, 84, 153, 255)
	ub := gocv.NewScalar(80, 255, 255, 255)
	hsv := gocv.NewMat()
	mask := gocv.NewMat()
	gocv.CvtColor(img, &hsv, gocv.ColorBGRToHSV)
	gocv.InRangeWithScalar(hsv, lb, ub, &mask)
	win := gocv.NewWindow("hi")
	win.IMShow(mask)
	win.WaitKey(0)
}
```

### 替换绿幕
```go
package main

import (
	"fmt"
	
	"gocv.io/x/gocv"
	"image"
)

// CreateImg create a solid image based on params
func CreateImgByBGR(sizex int, sizey int, b float64, g float64, r float64) gocv.Mat {
	img := gocv.NewMatWithSizeFromScalar(gocv.NewScalar(b, g, r, 255), sizex, sizey, gocv.MatTypeCV8UC3)
	return img
}

func Convert(srcPath string, dstPath string, r, g, b float64) {

}



func main() {
	lb := gocv.NewScalar(68, 84, 153, 255)
	ub := gocv.NewScalar(80, 255, 255, 255)

	hsv := gocv.NewMat()
	defer hsv.Close()
	mask := gocv.NewMat()
	defer mask.Close()
	mask_inv := gocv.NewMat()
	defer mask.Close()
	frame := gocv.NewMat()
	defer frame.Close()
	person := gocv.NewMat()
	defer frame.Close()
	kernel := gocv.GetStructuringElement(gocv.MorphRect, image.Pt(3, 3))
	defer kernel.Close()
	//result := gocv.NewMat()
	//defer frame.Close()


	capt, err := gocv.VideoCaptureFile("video.mp4")
	defer capt.Close()
	if err != nil {
		panic(err)
	}

	//count := capt.Get(gocv.VideoCaptureFrameCount)

	fps := capt.Get(gocv.VideoCaptureFPS)
	width := int(capt.Get(gocv.VideoCaptureFrameWidth))
	height := int(capt.Get(gocv.VideoCaptureFrameHeight))
	writer, err := gocv.VideoWriterFile("output.mp4", "avc1", fps, width, height, true)
	defer writer.Close()

	if err != nil {
		panic(err)
		return
	}

	for {
		if ok := capt.Read(&frame); ok {
			gocv.CvtColor(frame, &hsv, gocv.ColorBGRToHSV)
			gocv.InRangeWithScalar(hsv, lb, ub, &mask)

			gocv.BitwiseNot(mask, &mask_inv)
			gocv.Erode(mask_inv, &mask, kernel)
			gocv.BitwiseAndWithMask(frame, frame, &person, mask_inv)

			err = writer.Write(person)
			if err != nil {
				fmt.Printf("err occur when write frame: %s", err)
			}
		} else {
			break
		}
	}


}

```
