# 介绍 #

矢量类对于基于3维空间的数值计算、模拟以及动画非常有用，是整个空间计算的数学基础。
本矢量类实现了矢量的各种计算法则：
```
+    矢量加法
-    矢量减法
*    矢量乘法（与矢量则是点乘，与数值则是尺度放大）
/    矢量除法（只有数值输入，尺度缩小）
^    矢量叉乘（用于求垂直矢量）
```

同时提供了两个属性的获得：
  1. length：求矢量得长度
  1. normal：求矢量的单位矢量

# 代码 #
```
/*****************************************************************************
 * 通用数学矢量类算法封装
 *
 * 本代码基于GPL协议发布
 *****************************************************************************/
#ifndef __VECTOR_H__
#define __VECTOR_H__

#include <math.h>

class Vector
{
public:
	float x, y, z;

	// constructors

	Vector()
	{
		this->x = 0;
		this->y = 0;
		this->z = 0;
	}

	Vector(float x, float y, float z)
	{
		this->x = x;
		this->y = y;
		this->z = z;
	}

	Vector operator =(Vector v)
	{
		this->x = v.x;
		this->y = v.y;
		this->z = v.z;

		return *this;
	}

	// operators

	Vector operator +(Vector v)
	{
		return Vector(this->x + v.x,
			      this->y + v.y,
			      this->z + v.z);
	}

	Vector operator +=(Vector v)
	{
		return (*this = *this + v);
	}

	Vector operator -(Vector v)
	{
		return Vector(this->x - v.x,
			      this->y - v.y,
			      this->z - v.z);
	}

	Vector operator -=(Vector v)
	{
		return (*this = *this - v);
	}

	Vector operator *(float scale)
	{
		return Vector(this->x * scale,
			      this->y * scale,
			      this->z * scale);
	}

	Vector operator *=(float scale)
	{
		return (*this = *this * scale);
	}

	Vector operator *(Vector v)
	{
		return Vector(this->x * v.x,
			      this->y * v.y,
			      this->z * v.z);
	}

	Vector operator *=(Vector v)
	{
		return (*this = *this * v);
	}

	Vector operator ^(Vector v)
	{
		return Vector(this->y * v.z - this->z * v.y,
			      this->z * v.x - this->x * v.z,
			      this->x * v.y - this->y * v.x);
	}

	Vector operator ^=(Vector v)
	{
		return (*this = *this ^ v);
	}

	Vector operator /(float divider)
	{
		return Vector(this->x / divider,
			      this->y / divider,
			      this->z / divider);
	}

	Vector operator /=(float divider)
	{
		return (*this = *this / divider);
	}

	// attributes and transisions

	float length()
	{
		return sqrt(x * x + y * y + z * z);
	}

	Vector normal()
	{
		return Vector(x / length(),
			      y / length(),
			      z / length());
	}
};

#endif
```