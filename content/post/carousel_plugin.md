---
title: "轮播图插件"
date: 2018-03-01T08:43:12+08:00
categories:
- technique
- plugin
tags:
- jquery
- plugin
keywords:
- carousel
thumbnailImage: /img/carousel.png
---

<!--more-->  
![](/img/carousel.png)  

心血来潮,写了一个简单的轮播图插件,带有左右切换功能、底部导航功能、自动循环功能。代码如下:  

{{< codeblock "carousel.js" "js" "" "" >}}(function () {
    $.Carousel = function (elem, options) {
        var curr = this;
        var contanier, navContanier, prevElem, nextElem, imgArry;
        var curIndex = 0;
        var autoTimeout = 0;

        curr.init = function () {
            curr.options = $.extend({}, $.Carousel.defaultOptions, options);
            contanier = $(curr.options.contanier);
            navContanier = $(curr.options.nav);
            prevElem = $(curr.options.prev);
            nextElem = $(curr.options.next);
            imgArry = curr.options.images;

            if (imgArry.length != 0) {
                var imgHtml = "", navHtml = "";
                //为解决收尾图片衔接问题,故在首尾两侧克隆一份首尾的图片以做过渡
                imgHtml += '<li class="mLastClone" style="background-image:url(' + imgArry[imgArry.length - 1] + ');opacity:0;"></li>';
                for (var i = 0; i < imgArry.length; i++) {
                    imgHtml += '<li style="background-image:url(' + imgArry[i] + ');opacity:0;"></li>';
                    navHtml += '<li></li>';
                }
                imgHtml += '<li class="mFirstClone" style="background-image:url(' + imgArry[0] + ');opacity:0;"></li>';
                contanier.html(imgHtml);
                navContanier.html(navHtml);

                // 默认显示第一张图
                contanier.children().eq(curIndex + 1).css("opacity", 1);
                navContanier.children().removeClass(curr.options.navActiveClass);
                navContanier.children().eq(curIndex).addClass(curr.options.navActiveClass);

                // 默认自动播放
                if (curr.options.auto) {
                    curr.startAuto();
                    //鼠标进入容器时停止自动播放,离开时继续自动播放
                    $(curr.options.contanier + "," + curr.options.nav + "," + curr.options.prev + "," + curr.options.next).hover(function () {
                        curr.stopAuto();
                    }, function () {
                        curr.startAuto();
                    });
                }

            }

            // 下一张
            nextElem.click(function () {
                curr.next();
            });

            // 上一张
            prevElem.click(function () {
                curr.prev();
            });

            // 跳转
            navContanier.children().click(function () {
                if ($(this).hasClass(curr.options.navActiveClass)) {
                    return;
                }
                curr.index($(this).index());
            });

        };

        curr.reset = function () {
            contanier.children().css("opacity", 0);
            contanier.children().css("left", 0);
            navContanier.children().removeClass(curr.options.navActiveClass);
        };

        curr.next = function () {
            curr.reset();
            contanier.children().eq(curIndex + 1).css("left", "-100%");
            curIndex++;
            if (curIndex == imgArry.length) {
                curIndex = 0;
            }
            contanier.children().eq(curIndex + 1).css("opacity", 1);
            navContanier.children().eq(curIndex).addClass(curr.options.navActiveClass);
        };

        curr.prev = function () {
            curr.reset();
            contanier.children().eq(curIndex + 1).css("left", "100%");
            if (curIndex == 0) {
                curIndex = imgArry.length;
            }
            curIndex--;
            contanier.children().eq(curIndex + 1).css("opacity", 1);
            navContanier.children().eq(curIndex).addClass(curr.options.navActiveClass);
        };

        curr.index = function (index) {
            curr.reset();
            if (curIndex > index) {
                contanier.children().eq(curIndex + 1).css("left", "100%");
            } else {
                contanier.children().eq(curIndex + 1).css("left", "-100%");
            }
            curIndex = index;
            contanier.children().eq(curIndex + 1).css("opacity", 1);
            navContanier.children().eq(curIndex).addClass(curr.options.navActiveClass);
        };

        curr.startAuto = function () {
            autoTimeout = setTimeout(() => {
                curr.next();
                curr.startAuto();
            }, curr.options.intervals);
        };

        curr.stopAuto = function () {
            clearTimeout(autoTimeout);
        };

        curr.init();
    };

    $.Carousel.defaultOptions = {
        navActiveClass: "active",
        intervals: 2000,
        auto: true
    };

    $.fn.Carousel = function (options) {
        return this.each(function () {
            (new $.Carousel(this, options));
        });
    };
})();
{{< /codeblock >}}

{{< codeblock "carousel.css" "css" "" "" >}}* {
    box-sizing: border-box;
}

body {
    margin: 0;
    padding: 0;
}

ul {
    padding: 0;
    margin: 0;
}

ul>li {
    list-style: none;
}

.carousel-container {
    position: relative;
    overflow: hidden;
    width: 1200px;
    margin: auto;
    height: 600px;
}

.carousel-list {
    clear: both;
    position: relative;
    width: 100%;
    height: 100%;
}

.carousel-container:hover .carousel-arrow {
    visibility: visible;
    opacity: 1;
}

.carousel-list>li {
    position: absolute;
    width: 100%;
    height: 100%;
    top: 0;
    left: 0;
    z-index: 1;
    background-position: center;
    background-repeat: no-repeat;
    background-size: cover;
    transition: left .3s ease-in-out, opacity .3s ease-in-out;
    font-size: 72px;
    color: #fff;
    text-align: center;
    line-height: 600px;
}

.carousel-arrow {
    position: absolute;
    top: 50%;
    cursor: pointer;
    transform: translateY(-50%);
    transition: background-color .3s ease-in-out, opacity .3s ease-in-out;
    z-index: 999;
    padding: 5px;
    visibility: hidden;
    opacity: 0;
}

.carousel-arrow:hover {
    background-color: rgba(0, 0, 0, 0.5);
}

.carousel-arrow.left {
    left: 20px;
}

.carousel-arrow.right {
    right: 20px;
}

.carousel-nav {
    clear: both;
    position: absolute;
    bottom: 20px;
    left: 50%;
    transform: translateX(-50%);
    z-index: 999;
}

.carousel-nav>li {
    float: left;
    width: 10px;
    height: 10px;
    border-radius: 50%;
    background-color: #fff;
    transition: background-color .3s ease-in-out;
    cursor: pointer;
}

.carousel-nav>li:hover {
    background-color: #72c5ea;
}

.carousel-nav>li.active {
    background-color: #49b8ea;
}

.carousel-nav>li+li {
    margin-left: 10px;
}  
{{< /codeblock >}}

{{< codeblock "carousel.html" "xml" "" "" >}}<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <link rel="stylesheet" href="./css/carousel.css">
    <title>轮播图</title>
</head>

<body>
    <div class="carousel-container mCarousel">
        <ul class="carousel-list mCarouselContent"></ul>
        <div class="carousel-arrow left mPrevBtn">
            <img src="./images/left.png" alt="">
        </div>
        <div class="carousel-arrow right mNextBtn">
            <img src="./images/right.png" alt="">
        </div>
        <ul class="carousel-nav mCarouselNav"></ul>
    </div>

    <script src="./js/jquery-3.3.1.js"></script>
    <script src="./js/carousel.js"></script>
    <script>
        $(".mCarousel").Carousel({
            contanier: ".mCarouselContent",
            nav: ".mCarouselNav",
            prev: ".mPrevBtn",
            next: ".mNextBtn",
            navActiveClass: "active",
            images: ["./images/img1.jpg", "./images/img2.jpg", "./images/img3.jpg"],
            intervals: 3000,
            auto: true
        });
    </script>
</body>

</html>
{{< /codeblock >}}
