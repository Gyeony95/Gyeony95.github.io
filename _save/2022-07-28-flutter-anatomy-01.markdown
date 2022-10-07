---
layout: post
title: 플러터 해부학 01. 플러터 애니메이션을 파헤쳐보자!
image: flutter_logo.png
tags: 플러터 animation curve tween
categories: flutter
---
나는 안드로이드 개발을 하다가 플러터로 넘어왔지만 항상 느끼는점은 화면 그리기가 너무너무 간단하다는 것이다.
하지만 더 화려한 UI를 만들기 위해서는 화면이 동적으로 움직이는 애니메이션이 필수라고 생각한다.
이번 포스팅에서는 기본적으로 애니메이션을 어떻게 그리고 어떤식으로 커스텀할 수 있는지에 대해 알아보려고 한다.

***

#### 01. 기초 애니메이션 사용 방법

###### 1-1. AnimationController

애니메이션의 기본요소중의 하나인 애니매이션 컨트롤러이다.
모든 애니메이션은 이 컨트롤러를 통해 실행되고 상태를 얻어올 수 있다!

{% highlight markdown %}
late final AnimationController controller;

@override
void initState() {
    controller = AnimationController(vsync: this, duration: const Duration(seconds: 1));
}

@override
void dispose() {
    super.dispose();
    controller.dispose();
}
{% endhighlight %}

위 코드는 애니메이션 컨트롤러를 정의하고, 실행시는 1초간 수행하겠다는 의미이다.
메모리 관리를 위해 화면 dispose 시에는 controller 또한 dispose 해줘야 하는것을 잊지말자!

###### 1-2. Animation

Tween으로 초기화 하며, 애니메이션을 어디부터 어디까지 어떤 형태로 수행할것인지에 대해 정의한다.

{% highlight markdown %}
late Animation<Offset> animation;

@override
void initState() {
    topAnimation = Tween<Offset>(begin: const Offset(0,0), end: const Offset(0,-1))
        .animate(CurvedAnimation(parent: controller, curve: Curves.easeInCubic));
}
{% endhighlight %}

animation 변수를 초기화
begin과 end는 각각 시작점과 끝점을 의미하며 현재의 포지션을 x = 0, y = 0 이라고 치고 1만큼 움직이면 위젯의 크기만큼 움직인다는 의미이다.  
그렇기때문에 위 코드는 현재위치에서 위젯의 높이만큼 위로 움직인다는 의미이고 애니메이션을 실행하고 나면 위젯은 위로 올라가 보이지 않게된다 ㅎㅎ

###### 1-3. 실행 대상 지정하기

위에서 애니메이션을 어떤 형태로 언제동안 실행할지에 대해 정의를 했기떄문에 이제는 애니메이션을 적용할 대상에 대해 지정을 할때다.  
위젯을 여러가지 Transiton으로 감쌀수 있는데 우리는 SlideTransition 이라는 위젯으로 감싸 슬라이드 애니메이션을 구현할것이다.

{% highlight markdown %}
  Widget topContainer(){
    return SlideTransition(
      position: topAnimation,
      child: Container(
        height: 150,
        decoration: const BoxDecoration(
            color: Colors.lightBlue,
            borderRadius: BorderRadius.vertical(bottom: Radius.circular(40),)
        ),
      ),
    );
  }
{% endhighlight %}

위와 같은 방법으로 SlideTransition 위젯으로 감싸준 후 position 값에 앞에서 정의한 topAnimation을 적용하여 애니메이션을 실행할 준비를 마친다.

###### 1-4. 전체 코드

{% highlight markdown %}
import 'dart:math';

import 'package:flutter/material.dart';

class AnimationExample extends StatefulWidget{
  const AnimationExample({Key? key}) : super(key: key);

  @override
  State<AnimationExample> createState() => _AnimationExampleState();
}

class _AnimationExampleState extends State<AnimationExample>
    with TickerProviderStateMixin{

  bool isRun = false;

  // 상단 컨테이너 애니메이션
  late AnimationController controller;
  late Animation<Offset> topAnimation;

  // 가운데 컨테이너 애니메이션
  late Animation<Offset> middleAnimation;

  @override
  void initState() {
    super.initState();
    controller = AnimationController(vsync: this, duration: const Duration(seconds: 1));
    topAnimation = Tween<Offset>(begin: const Offset(0,0), end: const Offset(0,-1))
        .animate(CurvedAnimation(parent: controller, curve: Curves.easeInCubic));

    middleAnimation = Tween<Offset>(begin: const Offset(0,0), end: const Offset(1,0))
        .animate(CurvedAnimation(parent: controller, curve: Curves.easeInCubic));
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('애니메이션 예제'),),
      body: Stack(
        children: [
          Column(
            children: [
              topContainer(),
              middleContainer(),
              bottomContainer(),
            ],
          ),
          Positioned(
              bottom: 50,
              right: 20,
              child: runButton())
        ],
      ),
    );
  }

  Widget topContainer(){
    return SlideTransition(
      position: topAnimation,
      child: Container(
        height: 150,
        decoration: const BoxDecoration(
            color: Colors.lightBlue,
            borderRadius: BorderRadius.vertical(bottom: Radius.circular(40),)
        ),
      ),
    );
  }

  Widget middleContainer(){
    return SlideTransition(
      position: middleAnimation,
      child: Container(
        margin: const EdgeInsets.only(top: 20, left: 10),
        height: 100,
        child: ListView.builder(
          itemCount: 10,
          scrollDirection: Axis.horizontal,
          itemBuilder: (context, index) => Container(
            width: 100,
            height: 100,
            color: Colors.primaries[Random().nextInt(Colors.primaries.length)],
          ),
        ),
      ),
    );
  }

  Widget bottomContainer(){
    return Expanded(
      child: ListView.builder(
        itemCount: 10,
        itemBuilder: (context,index) {
          Animation<double> itemAnimation;
          itemAnimation = Tween<double>(begin: 1, end: 0)
              .animate(CurvedAnimation(parent: controller, curve: Interval(0.1 * index, 0.2 + index * 0.1)));
          return FadeTransition(
            opacity: itemAnimation,
            child: Container(
              margin: const EdgeInsets.all(10),
              decoration: const BoxDecoration(
                color: Colors.orangeAccent,
                borderRadius: BorderRadius.all(Radius.circular(10)),
              ),
              child: const SizedBox(
                width: double.infinity,
                height: 70,
              ),
            ),
          );
        },
      ),
    );
  }

  Widget runButton(){
    return GestureDetector(
      onTap: (){
        if(isRun){
          controller.reverse();
        }else{
          controller.forward();
        }
        isRun = !isRun;
      },
      child: Container(
        width: 100,
        height: 50,
        color: Colors.amberAccent,
        alignment: Alignment.center,
        child: const Text('RUN!!', style: TextStyle(fontSize: 20),),
      ),
    );
  }
}

{% endhighlight %}

클래스 내부에는 처럼 Scaffold 하위에 애니메이션을 적용시킬 위젯들과 애니메이션을 실행시켜줄 runButton()을 정의해준다.  
그리고 'RUN!!' 버튼을 누르면 미리 정의해둔 애니메이션이 실행된다.

[<img src="{{site.baseurl}}/images/01_animation_example_result.gif" width="250"/>]({{site.baseurl}}/images/01_animation_example_result.gif)


