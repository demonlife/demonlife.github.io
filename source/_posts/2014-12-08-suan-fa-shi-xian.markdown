---
layout: post
title: "算法实现"
date: 2014-12-08 08:47:25 +0800
comments: true
categories: 
---


<div id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#sec-1">1 二分查找</a></li>
</ul>
</div>
</div>

<div id="outline-container-1" class="outline-2">
<h2 id="sec-1">二分查找</h2>
<div class="outline-text-2" id="text-1">

<ul>
<li>非递归的代码
</li>
</ul>


{% codeblock lang:c %}
    int search(int array[], int n, int v) {
        int left, right, middle;
        left = 0, right = n - 1;
        while (left <= right) {
            middle = (left + right) / 2;
            if (array[middle] > v) {
                right = middle;
            } else if (array[middle] < v) {
                left = middle;
            } else {
                return middle;
            }
        }
        return -1;
    }
{% endcodeblock %}
<ul>
<li>递归代码
</li>
</ul>


{% codeblock lang:c %}
    int search(int array[], int low, int high, int key) {
        int mid = (low + high) / 2;
        if (low > high) 
            return -1;
        if (array[mid] == key)
            return mid;
        else if (array[mid] > key)
            return search(array, low, mid-1, key);
        else
            return search(array, mid+1, high, key);
    }
{% endcodeblock %}
<p>
  以上两种方法中middle的计算方法可能会出现问题，比如当low+high的值超出了low/high类型
  的表示范围，则middle的值将不准确，保险的做法是: mid = low + (high - low) / 2
</p><ul>
<li>另一种解法
</li>
</ul>


{% codeblock lang:c %}
    int search(int array[], int n, int v) {
        int left, right, mid;
        left = -1, right = n;
        while(left + 1 != right) {
            mid = left + (right - left) / 2;
            if(array[mid] < v) {
                left = mid;
            } else {
                right = mid;
            }
        }
        if (right >= n || array[right] != v)
            right = -1;
        return right;
    }
{% endcodeblock %}
</div>
</div>
