---
title: 'How to create an Android game'
date: 2022-08-01
permalink: /posts/2021/08/android
tags:
  - tutorial
  - android
---
Since 2014 I am part of the computer science summer camp in Jena. And all the time I have worked with the Android Studio, creating Android apps. Although the camp is the only time in the year I am creating apps, I have gained some experience. The most complicated app I have developed is a game, since you have to deal with different Activities and a Game-Loop. In this game you have to click a bit. This Bit occurs after a random time. At the end the game is kind of a reaction test. All source files and the images for the background, vit and the catcher can be found at https://github.com/julien-klaus/Simple-Android-Reaction-Game. 

To create the app you first need a `MainActivity`.

``````
package com.example.simplegame;

import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import android.widget.TextView;

import androidx.activity.ComponentActivity;

public class MainActivity extends ComponentActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        this.setContentView(R.layout.activity_main);

        this.findViewById(R.id.btn_start).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent i = new Intent(v.getContext(), GameActivity.class);
                startActivity(i);
            }
        });
    }

    @Override
    protected void onResume() {
        super.onResume();
        Intent i = getIntent();
        int latency = i.getIntExtra("latency", -1);
        if (latency > 0) {
            TextView tv_msg = (TextView) findViewById(R.id.tv_msg);
            tv_msg.setText(String.format("You have clicked the Bit in %d ms.", latency));
        }
    }
}
``````

As you can see this `Activity` creates two methods: `onCreate` and `onResume`. The first one is called when the app starts for the first time. The second one if you come back to the `MainActivity` from another `Activity`. In our case the other `Activity` is the game. So we get the time we have needed to click and print this time (its stored in the variable `latency`).

The second line in the `onCreate` method sets the view a beforehand designed layout `activity_main`. Android Studio allows you to design those layouts using an internal editor. But you can also write them by hand. In our case the layout has a `TextView` and a `Button`. The `LinearLayouts` are used for padding.

```````
<?xml version="1.0" encoding="utf-8"?>

<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:scrollbarAlwaysDrawVerticalTrack="false">

    <ImageView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_gravity="top"
        android:scaleType="centerCrop"
        android:src="@drawable/background" />

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:gravity="center_vertical"
        android:orientation="vertical">

        <TextView
            android:id="@+id/tv_msg"
            android:layout_width="match_parent"
            android:layout_height="0dp"
            android:layout_weight="0.5"
            android:gravity="center"
            android:text="Start a new game!"
            android:textSize="10pt"
            tools:text="Start a new game!" />

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="0dp"
            android:layout_weight="0.2"
            android:orientation="horizontal">

            <LinearLayout
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="0.2" ></LinearLayout>

            <Button
                android:id="@+id/btn_start"
                android:layout_height="wrap_content"
                android:layout_width="0dp"
                android:layout_weight="1"
                android:gravity="center"
                android:text="Start Game" />

            <LinearLayout
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="0.2" ></LinearLayout>

        </LinearLayout>

        <LinearLayout
            android:layout_width="wrap_content"
            android:layout_height="120dp"
            android:layout_gravity="bottom">

            <ImageView
                android:layout_width="80dp"
                android:layout_height="100dp"
                android:layout_gravity="left"
                android:scaleType="centerCrop"
                android:src="@drawable/runner"></ImageView>


            <ImageView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:src="@drawable/bit"></ImageView>

        </LinearLayout>

    </LinearLayout>

</FrameLayout>
```````

An important idea is the usage of the `android:layout_weight`. This allows you to give a percentage of the size of the view element. For example, I want the button to use $60\%$ of width of the app. So I add two `LinearLayouts`, with $20\%$ on each side of the `Button`.

As you have seen in the `onCreate` method of the `MainActivity` we start an `Activity` called `GameActivity`. This is the main game, where you click the bit. This `Activity` again has a `onCreate` method. That creates a `GamePanel` and uses this as the view object. You will see the definition in a while. Also it implements a `OnTouchListener`. This allows the `GameActivty` to handle a click event. **Note** as you can see in `onTouch` an `Intent` is used to switch between the activities. In contrast to the `MainActivity` this `Intent` puts the latency of the click as an extra. This allows the `MainActivity` to get this value.

```````
package com.example.simplegame;

import android.content.Intent;
import android.graphics.Point;
import android.os.Bundle;
import android.util.Log;
import android.view.MotionEvent;
import android.view.View;

import androidx.activity.ComponentActivity;

public class GameActivity extends ComponentActivity implements View.OnTouchListener {

    private GamePanel gamePanel;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        this.gamePanel = new GamePanel(this);
        this.gamePanel.setOnTouchListener(this);
        setContentView(this.gamePanel);
    }

    @Override
    public boolean onTouch(View view, MotionEvent motionEvent) {
        Point r = new Point((int)motionEvent.getX(), (int)motionEvent.getY());
        Log.i("debug", "click");
        int latency = this.gamePanel.getThread().check_clicked(r);
        if (latency > 0){
            Intent i = new Intent(this.getBaseContext(), MainActivity.class);
            i.putExtra("latency", latency);
            startActivity(i);
        }
        return false;
    }
}
```````

As promised now the `GamePanel`. This class implements a `SurfaceHolder`, that is used for drawing the bit in the game loop. The other task of the `GamePanel` is to start the `GameThread`. 

``````
package com.example.simplegame;

import android.content.Context;
import android.util.Log;
import android.view.SurfaceHolder;
import android.view.SurfaceView;

public class GamePanel extends SurfaceView implements SurfaceHolder.Callback {

    private GameThread game;

    public GamePanel(Context context){
        super(context);
        this.game = new GameThread(getHolder(), this);
        getHolder().addCallback(this);
        setFocusable(true);
    }

    public GameThread getThread(){
        return this.game;
    }

    @Override
    public void surfaceCreated(SurfaceHolder surfaceHolder) {
        game.start();
    }

    @Override
    public void surfaceChanged(SurfaceHolder surfaceHolder, int i, int i1, int i2) {
        if (this.getThread().isInterrupted()){
            Log.i("debug", "thread is interrupted");
        }
    }

    @Override
    public void surfaceDestroyed(SurfaceHolder surfaceHolder) {
        this.game.setRunning(false);
    }
}
``````
The `GameThread` contains the actual game logic. First it initialized a random time, that is counted down in each run of the loop. If the time is zero, the bit is drawn onto the surface. If there is a click onto the surface the bit itself checks if it was hit. If so, the thread is interrupted and the latency is returned.

``````
package com.example.simplegame;

import android.graphics.Bitmap;
import android.graphics.BitmapFactory;
import android.graphics.Canvas;
import android.graphics.Point;
import android.graphics.Rect;
import android.util.Log;
import android.view.SurfaceHolder;

import java.util.Random;

public class GameThread extends Thread {
    /*
    Class for the game mechanic.
    In running the action happens.
     */
    public final static int SLEEP = 25;
    public int GAME_WIDTH;
    public int GAME_HIGHT;

    private SurfaceHolder surfaceHolder;
    private GamePanel gamePanel;

    private Bitmap background;
    private Bit bit;

    private boolean running = true;

    private int latenz = 0;

    public GameThread(SurfaceHolder surfaceHolder, GamePanel gamePanel){
        super();
        this.surfaceHolder = surfaceHolder;
        this.gamePanel = gamePanel;
        background = BitmapFactory.decodeResource(gamePanel.getResources(), R.drawable.background);
    }

    public void setRunning(boolean running){
        this.running = running;
    }

    public Bit getBit (){
        return this.bit;
    }

    public int check_clicked(Point rect){
        if (this.bit.getRect() != null) {
            Log.i("debug", rect.toString());
            Log.i("debug", this.bit.getRect().toString());
            Log.i("debug", ""+this.bit.getRect().contains(rect.x, rect.y));
            if (this.bit.getRect().contains(rect.x, rect.y)) {
                this.bit.click();
                return this.latenz;
            }
        }
        return -1;
    }

    @Override
    public void run(){
        this.running = true;
        Random random = new Random();
        int time = random.nextInt(2000) + 1000;
        Log.i("debug", String.format("time to appear %d ms", time));
        Canvas canvas = this.surfaceHolder.lockCanvas();
        this.GAME_WIDTH = canvas.getWidth();
        this.GAME_HIGHT = canvas.getHeight();
        surfaceHolder.unlockCanvasAndPost(canvas);
        this.bit = new Bit(this.gamePanel);
        this.latenz = 0;
        while (this.running){
            canvas = this.surfaceHolder.lockCanvas();
            if (canvas != null){
                Rect backgroundRect = new Rect(0,0,this.GAME_WIDTH, this.GAME_HIGHT);
                canvas.drawBitmap(this.background, null, backgroundRect, null);
                if (time <= 0){
                    this.bit.draw(canvas);
                    this.bit.swing();
                    latenz += SLEEP;
                } else{
                    Log.i("debug", String.format("time to appear %d ms", time));
                    time -= SLEEP;
                }
                if (this.bit.is_clicked()){
                    this.setRunning(false);
                }
                surfaceHolder.unlockCanvasAndPost(canvas);
            }
            try{
                sleep(SLEEP);
            } catch (InterruptedException ie){
                ie.printStackTrace();
            }
            if (!this.running){
                Log.i("debug", "reached");
                try{
                    this.join();
                } catch(Exception e){
                    e.printStackTrace();
                } finally{
                    Thread.currentThread().interrupt();
                }
            }
        }
    }
}
``````

There is one final class you have to know. It is the `Bit`. This class just take care of a single bit, by generating the random possition, swing the bit around, so it moves a little bit and checks if it is clicked.

``````
package com.example.simplegame;

import android.graphics.Bitmap;
import android.graphics.BitmapFactory;
import android.graphics.Canvas;
import android.graphics.Rect;
import android.util.Log;

import java.util.Random;

public class Bit{

    public Bitmap bit;
    private GamePanel game;

    private Rect rect;

    private static final int BIT_WIDTH = 200;
    private static final int BIT_HEIGHT = 100;

    private int x,y;

    private boolean clicked;

    public Bit(GamePanel game){
        this.game = game;
        this.bit = BitmapFactory.decodeResource(this.game.getResources(), R.drawable.bit);
        Log.i("debug", "Game width"+game.getWidth());
        Log.i("debug", "Game height"+game.getHeight());
        Random random = new Random();
        this.y = random.nextInt(game.getHeight()-this.BIT_HEIGHT);
        this.x = random.nextInt(game.getWidth()-this.BIT_WIDTH);
        this.rect = new Rect(this.x, this.y, this.x + BIT_WIDTH, this.y + Bit.BIT_HEIGHT);
        this.clicked = false;
    }

    public void swing(){
        int dx = 10;
        int dy = 10;
        // get random offset and scale it
        Random random = new Random();
        if (random.nextInt(2) == 1){
            dx = -10;
        }
        if (random.nextInt(2) == 1){
            dy = -10;
        }
        dx = dx * random.nextInt(5);
        dy = dy * random.nextInt(5);;
        // check bounds
        if (this.x + dx > this.game.getWidth() - this.BIT_WIDTH){
            dx = dx * -1;
        }
        if (this.y + dy > this.game.getHeight() - this.BIT_HEIGHT){
            dy = dy * -1;
        }
        if (this.x + dx < 0){
            dx = dx * -1;
        }
        if (this.y + dy < 0){
            dy = dy * -1;
        }
        // update position
        this.rect.offset(dx, dy);
        this.x = this.x + dx;
        this.y = this.y + dy;
    }

    public void draw(Canvas canvas){
        canvas.drawBitmap(this.bit, null, this.rect, null);
    }

    public Rect getRect(){
        return this.rect;
    }

    public boolean is_clicked(){
        return this.clicked == true;
    }

    public void click(){
        this.clicked = true;
    }
}
``````

After understanding how a single element, like the bit is implemented, and how the `GameThread` works, you can now implement many more objects, that appear on the surface. No matter if Flappy Bird, Mario or any other game, the key concepts are still the same. 

So install Android Studio and start your game development.


![AndroidStudio](/images/android/studio.png) 