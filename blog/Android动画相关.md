### 前言
在项目中遇到了这样一个需求，一个Imageview围绕另一个Imageview沿倾斜的椭圆路径旋转，具体效果如下：



![aa.gif](http://upload-images.jianshu.io/upload_images/6687414-f3371f9315cc0e23.gif?imageMogr2/auto-orient/strip)



### 思路
* 确定椭圆的坐标
* 用PathMeasure计算移动点的坐标
* 设置属性动画和差值器
* 添加属性动画监听
* 设置视图坐标

### 环境
1. API 25
2. JDK 1.8.0_101


### 涉及到的知识点

1. 获取控件在屏幕中的绝对坐标

   ```
   //获取控件当前位置
   int[] startLoc = new int[2];
   rotateIv.getLocationInWindow(startLoc);

   //获取被围绕控件的起始点
   int[] parentStart = new int[2];
   customIv.getLocationInWindow(parentStart);

   //获取被围绕坐标的终点
   int[] parentEnd = new int[2];
   parentEnd[0] = parentStart[0] + customIv.getWidth();
   parentEnd[1] = parentStart[1] + customIv.getHeight();
   ```

2. 绘制椭圆并旋转角度

   ```
    //构建椭圆
   //        KLog.i((parentStart[0]-deviation)+"   " +(parentStart[1]-deviation)+"   "+(parentEnd[0]+deviation)+"   "+(parentEnd[1]+deviation));
           Path path = new Path();
           RectF rectF = new RectF(parentStart[0]-120,parentStart[1]-220,parentEnd[0]+100,parentEnd[1]);//椭圆大小需自己调整
           path.addArc(rectF,0,360);

           //设置椭圆倾斜度数
           Matrix matrix = new Matrix();
           matrix.setRotate(-14,(parentStart[0]+parentEnd[0])/2,(parentStart[1]+parentEnd[1])/2);
           path.transform(matrix);
   ```

3. 视图加载监听

   ```
   //添加视图加载完成监听，getLocationInWindow需等到视图加载完成后才能返回正确值，否则为0
   customIv.getViewTreeObserver().addOnGlobalLayoutListener(new ViewTreeObserver.OnGlobalLayoutListener() {
       @Override
       public void onGlobalLayout() {
           // TODO Auto-generated method stub
           startRotate();
       }
   });
   ```

4. 属性动画监听

   ```
    //添加监听
           valueAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
               @Override
               public void onAnimationUpdate(ValueAnimator animation) {
                   //获取当前位置
                   float value = (float) animation.getAnimatedValue();
                   //传入一个距离distance(0<=distance<=getLength())，然后会计算当前距
                   // 离的坐标点和切线，pos会自动填充上坐标
                   pathMeasure.getPosTan(value,mCurrentPosition,null);
                   //打印当前坐标
   //                KLog.i(mCurrentPosition[0]+"    "+mCurrentPosition[1]);
                   //设置视图坐标
                   rotateIv.setX(mCurrentPosition[0]);
                   rotateIv.setY(mCurrentPosition[1]);
               }
           });
           
           valueAnimator.start();
   ```



### 所有代码

```
package cn.edu.neu.providence.activity;

import android.animation.ValueAnimator;
import android.app.Activity;
import android.graphics.Matrix;
import android.graphics.Path;
import android.graphics.PathMeasure;
import android.graphics.RectF;
import android.os.Bundle;
import android.view.View;
import android.view.ViewTreeObserver;
import android.view.Window;
import android.view.animation.AccelerateDecelerateInterpolator;
import android.view.animation.LinearInterpolator;
import android.widget.ImageView;

import com.socks.library.KLog;

import butterknife.BindView;
import butterknife.ButterKnife;
import cn.edu.neu.providence.R;

public class MainActivity extends Activity {

    @BindView(R.id.rotate_iv)
    ImageView rotateIv;
    @BindView(R.id.custom_iv)
    ImageView customIv;
    private float[] mCurrentPosition = new float[2];
    private PathMeasure pathMeasure;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        requestWindowFeature(Window.FEATURE_NO_TITLE);
        setContentView(R.layout.activity_main);
        ButterKnife.bind(this);
        //添加视图加载完成监听，getLocationInWindow需等到视图加载完成后才能返回正确值，否则为0
        customIv.getViewTreeObserver().addOnGlobalLayoutListener(new ViewTreeObserver.OnGlobalLayoutListener() {
            @Override
            public void onGlobalLayout() {
                // TODO Auto-generated method stub
                startRotate();
            }
        });
    }

    @Override
    protected void onResume() {
        super.onResume();
    }

    private void startRotate() {

        //获取控件当前位置
        int[] startLoc = new int[2];
        rotateIv.getLocationInWindow(startLoc);

        //获取被围绕控件的起始点
        int[] parentStart = new int[2];
        customIv.getLocationInWindow(parentStart);

        //获取被围绕坐标的终点
        int[] parentEnd = new int[2];
        parentEnd[0] = parentStart[0] + customIv.getWidth();
        parentEnd[1] = parentStart[1] + customIv.getHeight();

        //构建椭圆
//        KLog.i((parentStart[0]-deviation)+"   " +(parentStart[1]-deviation)+"   "+(parentEnd[0]+deviation)+"   "+(parentEnd[1]+deviation));
        Path path = new Path();
        RectF rectF = new RectF(parentStart[0]-120,parentStart[1]-220,parentEnd[0]+100,parentEnd[1]);//椭圆大小需自己调整
        path.addArc(rectF,0,360);

        //设置椭圆倾斜度数
        Matrix matrix = new Matrix();
        matrix.setRotate(-14,(parentStart[0]+parentEnd[0])/2,(parentStart[1]+parentEnd[1])/2);
        path.transform(matrix);

        //pathMeasure用来计算显示坐标
        pathMeasure = new PathMeasure(path,true);

        //属性动画加载
        ValueAnimator valueAnimator = ValueAnimator.ofFloat(0,pathMeasure.getLength());

        //设置动画时长
        valueAnimator.setDuration(10000);

        //加入差值器
        valueAnimator.setInterpolator(new LinearInterpolator());

        //设置无限次循环
        valueAnimator.setRepeatCount(ValueAnimator.INFINITE);

        //添加监听
        valueAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator animation) {
                //获取当前位置
                float value = (float) animation.getAnimatedValue();
                //boolean getPosTan(float distance, float[] pos, float[] tan) ：
                //传入一个距离distance(0<=distance<=getLength())，然后会计算当前距
                // 离的坐标点和切线，pos会自动填充上坐标
                pathMeasure.getPosTan(value,mCurrentPosition,null);
                //打印当前坐标
//                KLog.i(mCurrentPosition[0]+"    "+mCurrentPosition[1]);
                //设置视图坐标
                rotateIv.setX(mCurrentPosition[0]);
                rotateIv.setY(mCurrentPosition[1]);
            }
        });

        valueAnimator.start();
    }
}
```
