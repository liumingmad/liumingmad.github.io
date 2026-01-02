title: MarkDown语法学习
author: ming
date: 2018-05-08 22:37:22
tags:
---
# fuck what
## fuck what

*fuck what*

**fuck what**

***nini***

* 彪悍
* 彪悍了
* 要加空格


1. wha
2. nani
3. [google](https://www.google.com)
  
![](http://xianbai.me/learn-md/article/extension/images/code-js.png)


  
```java
  private void initPlayer(LessonDetailModel lessonDetailModel) {
        initPlayerView();
        LogUtils.i(mPlayer.getPlayerState() + ", pos=" + mPlayer.getCurrentPosition());
        if (mPlayer.isPlaying()) {
            mPlayer.setDisplay(mSurfaceView.getHolder());
        } else {
            requestVideoInfo(lessonDetailModel.getAudioList().get(0).getId());
        }
    }
```