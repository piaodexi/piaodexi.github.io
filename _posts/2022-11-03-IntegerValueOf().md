---
title: Integer.valueOf() 주의사항 
author: dexi
date: 2022-11-03 00:00:01 +0800
categories: [Blog, java]
tags: [java]
math: true
mermaid: true
futrue: true
published: true
---
서버에 방어로직을 작성 할 일이 있었다. 

특정 조건을 걸어주면 되는 거였는데..

<pre><code>
        if (result || (Integer.valueOf(mAuctionSettings.price1) 
                       == Integer.valueOf(mAuctionSettings.price2) 
                       && Integer.valueOf(mAuctionSettings.value1) 
                       == Integer.valueOf(mAuctionSettings.value2))){
            System.out.println("???valueOf???");
        }
</code></pre>

이런 로직이였는데 로그를 찍어봐도 이건 트루임에 틀림없다고 확신이 되는데 조건에 걸리지가 않았다.

가뜩이나 테스트 하기도 번거로워 2시간동안 애먼 삽질을 했다.. 

2시간의 변명을 써보자면 서버단이라 ftp에 배포하고 디비에서 테스트 데이터 셋팅하고 클라2개 켜고 하는 작업에 시간이 좀 걸린다.. 대략 30분정도..

여하튼 문제는 값의 범위였다. Integer.valueOf의 경우 -128 ~ 127 사이의 값만 받아서 연산 처리를 할 수 있는데 
그 이상의 값(ex 128이상)을 받아서 하지 못했던 것이다. 

<pre><code>
        if (result || 	  mAuctionSettings.price1
                       == mAuctionSettings.price2
                       && mAuctionSettings.value1
                       == mAuctionSettings.value2){
            System.out.println("????????????");
        }
</code></pre>

Integer.valueOf()를 쓰지 않고 처리하기로 했다.

값의 범위라 ... 대학교때 한창 문법 공부할때 이후로는 값의 범위는 생각을 않고 
타입만 염두에 두고 사용을 했는데...조심하게 될 계기가 되었다...



