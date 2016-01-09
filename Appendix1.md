# 介绍 #

PID算法是自动控制领域最常用的误差消除算法，可以用其将一个动态数据输入序列进行处理达到
自稳定的特性。在这里实现的是PID算法的核心（比例、积分、微分）的数值计算部分，采用模块
化数据结构使得该代码能够在一个项目中完成多个PID控制任务的实现。

# 代码 #

```
/*****************************************************************************
 * 通用PID算法
 *
 * 本代码基于GPL协议发布
 *****************************************************************************/
#include <stdio.h>
#include <math.h>

#define PID_TYPE float

// PID控制模块（每个控制单元需要一个此结构）
struct pid_unit {
	PID_TYPE pid_sens[2];	// 保存连续最新3个输入数据序列
	PID_TYPE pid_cpid[3];	// 比例、积分、微分系数
	PID_TYPE pid_dpid[3];	// PID变量
};

// 初始化PID模块的代码
void pid_init(struct pid_unit *unit,  // PID控制模块指针
              PID_TYPE p,             // 比例系数
              PID_TYPE i,             // 积分系数
              PID_TYPE d)             // 微分系数
{
	unit->pid_cpid[0] = p;
	unit->pid_cpid[1] = i;
	unit->pid_cpid[2] = d;

	unit->pid_dpid[0] = 0;
	unit->pid_dpid[1] = 0;
	unit->pid_dpid[2] = 0;

	unit->pid_sens[0] = 0;
	unit->pid_sens[1] = 0;
}

// PID算法控制函数，通过此函数实现对输入数据序列的PID处理
// 返回值：PID计算所得值
PID_TYPE pid_control(struct pid_unit *unit,   // PID控制模块
                     PID_TYPE input,          // 当前输入数据
                     PID_TYPE time)           // 距离上个数据的时间间隔
{
	// adjust the FIFO preserving the sensor data
	unit->pid_sens[1] = unit->pid_sens[0];
	unit->pid_sens[0] = input;

	// calculate each pid variable
	unit->pid_dpid[0]  =  unit->pid_sens[0] * unit->pid_cpid[0];
	unit->pid_dpid[1] +=  unit->pid_sens[0] * unit->pid_cpid[1] * time;
	unit->pid_dpid[2]  = (unit->pid_sens[0] - unit->pid_sens[1])/ time *
			      unit->pid_cpid[2];

	return unit->pid_dpid[0] + 
	       unit->pid_dpid[1] +
	       unit->pid_dpid[2];
}

// 使用演示
int main()
{
	struct pid_unit unit;        // 生成一个PID控制模块
	pid_init(&unit, 0, 0, 1);    // 对该模块进行初始化

	int i;
	PID_TYPE input[1000];
	PID_TYPE output[1000];

        // 输入数据序列为一幅值为10的正弦波，输出是经过PID系数进行处理后的求和
	for(i=0; i<1000; i++) {
		input[i] = (PID_TYPE) 10 * sin((float)i/100);
		output[i] = pid_control(&unit, input[i], (float)1/100);
	}

        // 通过打印方式将输入、输出数据输出到控制台，通过管道进入文件后，可以用gnuplot
        // 等软件绘制成图标分析PID计算结果与输入对应关系
	for(i=0; i<1000; i++) {
		fprintf(stdout, "%8.4f\t%8.4f\t%8.4f\n", (float)i/100, input[i], output[i]);
	}

	return 0;
}


```