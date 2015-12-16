---
layout: post
title: 인스타그램 API 접근과 사진 불러오기
category: javascript
---

인스타그램에서 최근 내 사진들을 불러와 블로그나 사이트에 보여 주고자 하는데, 생각보다 은근 자세히 설명된 글을 찾기가 쉽지 않다. 대부분의 검색 결과에서는 인스타그램 피드 서비스를 제공해 주는 사이트만 나오지 실제 하드 코딩을 통한 API 접근 방식을 찾기가 어려웠다. 인스타그램 피드 서비스를 이용하는 것도 나쁘진 않지만 단점은 사진을 클릭했을 때 인스타그램 계정으로 이동하는것이 아니라 해당 서비스 사이트로 이동한다는 점이다. 이쁘지 않다.

그래서, 어차피 다양한 API를 이용해 보는것도 좋은 연습이고 해서 직접 시도해 보았고 그 결과를 공유하고자 한다.

각자 여러가지 상황이 있겠지만 크게 두 가지로 구분하자면 자신이 직접 서버를 운용하고 있어 서버측 코딩이 가능한 경우 (내 경우가 아니라 이 설명은 생략), 또는 나와 같이 서비스형 블로그를 이용하기에 서버측 코딩이 불가능한 경우.

서버측 코딩이 불가능하더라도 반드시 템플릿 수정을 통해 HTML, 자바스크립트 수정이 가능해야지만 한다. 또한,  AJAX 코딩 편의성을 위해 jQuery 라이브러리를 이용했음을 미리 밝힌다.

> 서버 컨트롤 없이 클라이언트측에서만 실행하는 것은 사실 추천하는 방법은 아니다. 인스타그램 API 문서에서도 안전성을 위해 서버측 코딩을 추천하고 있다.

### 클라이언트 아이디와 억세스 토큰
인스타그램 API를 통해 자신의 사진을 불러오기 위해서는 Access Tocken이란것이 필요하다. 그런데, 자신의 Access Tocken을 알기 위해선 Client ID가 필요하다.

1. 먼저, 인스타 그램 자신의 계정에 로그인 한 후, 자신의 아이디를 클릭하고 해당 화면 아래쪽에 API 페이지로 이동.
2. 상단 메뉴의 Manage Clients 메뉴를 통해 새로운 클라이언트 생성.
3. Client ID와 Redirect URI를 이용해 다음 주소로 이동.

```html
https://instagram.com/oauth/authorize/?client_id=CLIENT-ID&amp;redirect_uri=REDIRECT-URI&amp;response_type=token  
```

<blockquote>위 코드에서 CLIENT-ID와 REDIRECT-URI부분을 자신의 ID와 URI로 수정한다.</blockquote>

4. Authorise를 클릭하면 주소창의 쿼리를 통해 Access Tocken 반환 됨.

### HTML
실제 데이터는 AJAX를 이용할 것이므로 HTML로는 플레이스 홀딩만 해준다.

```
<div id="instaPics"></div>
```

### CSS

```css
#instaPics {  
    max-width: 320px;  
    overflow: hidden;  
}
.insta-box {  
    position: relative;  
    width: 120px;  
    height: 120px;  
    float: left;  
    margin: 4px;  
    border: 1px solid #ddd;  
}  
.image-layer {  
    overflow: hidden;  
    width: 100%;  
    height: 100%;  
}  
.image-layer img {  
    max-width: 100%;  
}  
.caption-layer {  
    display: none;  
    position: absolute;  
    top: 0;  
    background: rgba(255,255,255,0.8);  
    height: 100%;  
    width: 100%;  
    padding: 10px;  
    box-sizing: border-box;  
    font-size: 10px;  
    color: #333;  
}  
.insta-likes {  
    float: right;  
}
```

### JavaScript
> 아래 var tocken 함수에 위에서 얻은 Access Tocken을 입력 한다. 클라이언트 측 코딩을 비추천 하는 이유가 바로 이렇게 민감한 정보를 누구나 볼 수 있는 클라이언트 측에 저장해야 하기 때문 이다.

```javascript
jQuery(function($) {  
    var tocken = ""; /* Access Tocken 입력 */  
    var count = "6";  
    $.ajax({  
        type: "GET",  
        dataType: "jsonp",  
        cache: false,  
        url: "https://api.instagram.com/v1/users/self/media/recent/?access_token=" + tocken + "&count=" + count,  
        success: function(response) {  
         if ( response.data.length > 0 ) {  
              for(var i = 0; i < response.data.length; i++) {  
                   var insta = '<div class="insta-box">';  
                   insta += "<a target='_blank' href='" + response.data[i].link + "'>";  
                   insta += "<div class'image-layer'>";  
                   //insta += "<img src='" + response.data[i].images.thumbnail.url + "'>";  
                   insta += '<img src="' + response.data[i].images.thumbnail.url + '">';  
                   insta += "</div>";  
                   //console.log(response.data[i].caption.text);  
                   if ( response.data[i].caption !== null ) {  
                        insta += "<div class='caption-layer'>";  
                        if ( response.data[i].caption.text.length > 0 ) {  
                             insta += "<p class='insta-caption'>" + response.data[i].caption.text + "</p>"  
                        }  
                        insta += "<span class='insta-likes'>" + response.data[i].likes.count + " Likes</span>";  
                        insta += "</div>";  
                   }  
                   insta += "</a>";  
                   insta += "</div>";  
                   $("#instaPics").append(insta);  
              }  
         }  
         $(".insta-box").hover(function(){  
              $(this).find(".caption-layer").css({"backbround" : "rgba(255,255,255,0.7)", "display":"block"});  
         }, function(){  
              $(this).find(".caption-layer").css({"display":"none"});  
         });  
        }  
       });  
});  
```

### 참고
* <a href="https://instagram.com/developer/authentication/" target="_blank">인스타그램 API Authentication 문서</a>
