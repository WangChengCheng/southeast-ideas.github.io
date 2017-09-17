---
layout: post
title:  Java——Comparator
date:   2017-09-17 19:04:00 +0800
categories: Java
tag: Java Sort 排序
author: Michael Wang
source_id: 170917190400
---

* content
{:toc}

### 题目来源
CCF 201709试题第2题

### Code

```java
package test;

import java.util.ArrayList;
import java.util.Collections;
import java.util.Comparator;
import java.util.List;
import java.util.Scanner;

public class Event{
	private int keyNo;
	private int time;
	private int type; //Event Type: 0-->return, 1-->borrow
	
	public static void main(String[] args) {
		List<Event> lt = new ArrayList();
		Scanner sc = new Scanner(System.in);
		int N = sc.nextInt();
		int K = sc.nextInt();
		for(int i = 0; i < K; i++) {
			int w = sc.nextInt();
			int s = sc.nextInt();
			int c = sc.nextInt();
			Event event1 = new Event(w, s, 1);//Borrow As An Enevt
			Event event2 = new Event(w, s + c, 0);//Return As An Enevt
			lt.add(event1);
			lt.add(event2);
		}
		//Sort
		Collections.sort(lt, new Comparator<Event>() {
			@Override
			public int compare(Event o1, Event o2) {
				Event e1 = (Event) o1;
				Event e2 = (Event) o2;
				if (e1.getTime() > e2.getTime())
					return 1;
				else if(e1.getTime() == e2.getTime()) {
					if(e1.getType() > e2.getType())
						return 1;
					else if(e1.getType() == e2.getType()) {
						if(e1.getKeyNo() > e2.getKeyNo())
							return 1;
						else if(e1.getKeyNo() == e2.getKeyNo())
							return 0;
						else
							return -1;
					}
					else
						return -1;
				}
				else
					return -1;
			}
		});
		//OutPut
		List<Integer> stat = new ArrayList<>();
		for(int i = 1; i <= N; i++)
			stat.add(i);
		for(Event e: lt){
			if(e.getType() == 1) {//Borrow 借钥匙
				int ind = stat.indexOf(e.getKeyNo());//获得钥匙的位置
				stat.set(ind, -1);//将该位置置为-1
			}
			else if(e.getType() == 0) {//Return 还钥匙
				int ind = stat.indexOf(-1);
				stat.set(ind, e.keyNo);
			}
		}
		for(Integer s: stat)
			System.out.print(s + " ");
		
	}

	
	
	public Event(int keyNo, int time, int type) {
		this.keyNo = keyNo;
		this.time = time;
		this.type = type;
	}


	public int getKeyNo() {
		return keyNo;
	}

	public void setKeyNo(int keyNo) {
		this.keyNo = keyNo;
	}

	public int getTime() {
		return time;
	}

	public void setTime(int time) {
		this.time = time;
	}

	public int getType() {
		return type;
	}

	public void setType(int type) {
		this.type = type;
	}


}

	
```
