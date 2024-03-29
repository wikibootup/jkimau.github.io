---
layout: post
title: Backbone.js - Handlebars.js를 이용한 뷰 & 컨트롤러 디커플링
category: backbone.js
---

Backbone.js는 Angular.js, React.js에 이어 프론트 엔드 개발에 가장 많이 사용되는 MVC패턴(이라고 일단 하자. 정확히 말하자면 MVP도 아니고 MVVM도 아니고 그냥 MVW(Whatever)패턴 이다. 여튼, 이번 글 주제와 크게 연관 없으므로 이 부분은 패스)의 자바스크립트 프레임웍이다.

MVC패턴이 무조건 만능은 아니겠지만, 거의 대부분의 경우 모델, 뷰, 컨트롤러를 분리 시킴으로써 전체적으로 웹 앱의 유지 보수를 확실히 더 편하게 해준다. 특히 앱의 규모가 커질수록 MVC패턴의 역할은 더욱 중요해 진다.

이 글을 통해 Backbone이 가지고 있는 문제점과 Handlebars.js 템플릿 엔진을 통한 개선 방안을 얘기하고자 한다.

## DOM, jQuery 그리고 Backbone.js의 문제점
DOM(Document Object Model), 아주 쉽게 말하자면 HTML 등의 마크업 노드들을 트리 형태의 오브젝트로 만들어 접근, 수정 등을 용이하게 만들어 주는 것이다. MVC패턴 중 View에 해당 한다고 보면 된다. 그리고 모든 프론트엔드 개발의 근본적 목적은 원하는 데이터를(모델) 원하는 형태로 변형 또는 조합시켜서(컨트롤러) 화면에 보여지게(뷰) 하는 것이다. 따라서 이런 의미에서 DOM 접근은 어떤 프론트앤드 개발 또는 프론트앤드 프레임웍을 막론하고 필수적인 요소라 생각 한다.

다만, 어떻게 접근하느냐의 문제일 뿐이다.

jQuery는 이런 DOM 접근 및 수정에 있어 가장 많이 사용되는 라이브러리다. 그만큼 강력하고 사용하기 쉽다는 의미이기도 하다. Backbone의 경우 Angular와 달리 자체적으로 컨트롤러와 뷰가 바인드 되어 있지 않으므로 대부분은 jQuery 또는 이에 상응하는 다른 라이브러리를 이용해 DOM에 접근 한다.

문제는 여기서 시작된다.

애초에 MVC패턴이 추구하는 것이 모델, 뷰, 컨트롤러를 완전히 분리 시키자는 것인데 Backbone의 경우 컨트롤러에 해당하는 자바스크립트 파일에서 jQuery를 이용해 뷰에 직접 접근하게 된다. 이 점이 바로 Backbone.js를 정확히 MVC패턴이라 부르기 어렵게 하는 대목임과 동시에 내가 개인적으로 생각하는 Backbone.js의 가장 큰 약점이라 생각 한다.

간단한 뷰인 경우에는 짧게 끝나겠지만 많은 로직이 들어가고 복잡한 뷰 구조를 가지고 있으면 말이 달라진다. 더구나 시간이 지나면서 뷰의 업데이트가 추가 될수록 컨트롤러에 해당하는 자바스크립트 파일은 순식간에 jQuery로 인해 뷰와 바인드가 된 듯 안 된 듯한 어지러운 구조가 되어 버린다. 그렇게 강력하고 사용하기 편한 jQuery가 MVC패턴 관점에서 봤을 때 아주 성가신 골칫거리가 되는 순간이다.

이러한 이유로 특히 Angular.js 진영에서는 jQuery 사용 자체를 아주 꺼려하는 분위기(최근 몇년 사이에 벌어지고 있는 jQuery를 이대로 써도 되니 마니하는 것과는 특별이 연관이 없어 보이는)가 조금씩 확산 되고 있다. Angular.js 사용자 입장에서는 그럴만도 한 것이 컨트롤러와 뷰가 제대로 바인딩 되어 있어서 DOM 접근을 위한 라이브러리를 따로 사용하지 않아도 되는 경우가 대부분이다. 다만, 특히 dropdown 메뉴등의 간단한 애니메이션은 jQuery를 사용하는것이 훨씬 쉽게 끝날 일인데도 굳이 힘들게 외부 모듈까지 동원해가며 jQuery를 배척하는 모습이 조금은 안쓰러워 보이기도 하지만 어쨌든 MVC 패턴의 장점과 취지만을 놓고 보자면 충분히 이해가는 부분이기도 하다.

## Handlebars.js
Handlebars.js는 다양한 백엔드 또는 프론트엔드와 조합할 수 있는 템플릿 엔진이다. 특히 이런 템플릿 엔진들이 가져다 주는 가장 큰 이점은 DOM 컨트롤을 담당하는 많은 코드를 컨트롤러 파일에서 템플릿 파일로 옮길 수 있게 함으로써 컨트롤러를 깔끔히 유지하는데 큰 도움을 준다는 것이다. 컨트롤러를 깔끔하게 유지할 수 있다는 말은 곧 유지 보수 및 더욱 중요한 **기능 확장**에 큰 이점을 가져다 준다는 말과도 같다.
특히 Backbone.js등의 프레임웍과 같이 사용하면 서로의 장점이 더욱 부각되는 것 같다.

예제를 통해 템플릿 엔진을 사용할 때와 안할 때 컨트롤러와 뷰 파일의 차이를 살펴 보자.

#### Backbone without Handlebars

```html
<!-- index.html body 부분 -->
<div id="main">
    <ul class="car-list"></ul>
</div>
```

```javascript
// App.js
var Car = Backbone.Model.extend({
});

var car1 = new Car({brand: 'Honda', model: 'Jazz', year: '2012', color: 'black', is_secondhand: true});
var car2 = new Car({brand: 'Huyndai', model: 'Veloster', year: '2013', color: 'red', is_secondhand: false});
var car3 = new Car({brand: 'Kia', model: 'Sportage', year: '2015', color: 'white', is_secondhand: false});

var Cars = Backbone.Collection.extend({
    model: Car
});

var CarsView = Backbone.View.extend({
    template: 'index.html',
    el: '#main',
    initialize: function() {
        this.render();
    },
    render: function() {
        var carList = new Cars([car1, car2, car3]);
        var list = '';
        _(carList).each(function(car) {
            list += '<li>';
            list += car.get('brand') + ', ';
            list += car.get('model') + ', ';
            list += car.get('year') + ', ';
            list += car.get('color') + ', ';

            if (car.get('is_secondhand')) {
                list += ' (** Secondhand Car)';
            } else {
                list += ' (** New Car)';
            }

            list += '</li>';
        });

        this.$('ul.car-list').append(list);
    }
});

var TestView = new CarsView();
```
컨트롤러의 `render()` 함수내에서 jQuery를 이용해 뷰를 업데이트 한다는 것이 특징이다.

다음은 Handlebars.js 템플릿 엔진을 이용한 예제를 보자.


#### Backbone with handlebars

```html
<!-- index.html body 부분 -->
<div id="main">{% raw %}
    <ul class="car-list">
        {{#each car_list_array}}
            <li>
                {{brand}}, {{model}}, {{year}}, {{color}}
                {{#if is_secondhand}}
                    (** Secondhand Car)
                {{else}}
                    (** New car)
                {{/if}}
            </li>
        {{/each}}
    </ul>
{% endraw %}</div>
```

```javascript
var Car = Backbone.Model.extend({
});

var car1 = new Car({brand: 'Honda', model: 'Jazz', year: '2012', color: 'black', is_secondhand: true});
var car2 = new Car({brand: 'Huyndai', model: 'Veloster', year: '2013', color: 'red', is_secondhand: false});
var car3 = new Car({brand: 'Kia', model: 'Sportage', year: '2015', color: 'white', is_secondhand: false});

var Cars = Backbone.Collection.extend({
    model: Car
});

var CarsView = Backbone.View.extend({
    template: Handlebars.compile( $('#main').html() ),
    el: '#main',
    initialize: function() {
        this.render();
    },
    render: function() {
        var carList = new Cars([car1, car2, car3]);
        var carListArray = [];

        _(carList.models).each(function(car){
            carListArray.push(car.attributes);
        });

        this.$el.html(this.template({
            car_list_array: carListArray
        }));
    }
});

var TestView = new CarsView();
```
이전 예제와 비교해 봤을 때, 컨트롤러의 jQuery부분이 없어짐으로써 확연히 깔끔해진 것을 볼 수 있다.
이런 간단한 로직에서도 이정도의 차이를 낼 수 있다면 복잡한 로직에서는 그 위력이 더욱 발휘 된다.

## 결론
물론 Backbone을 다루다 보면 어쩔 수 없이 컨트롤러에서 DOM 접근이 불가피할 때가 분명 있다. 특히 위 예제는 다소 국한된 상황만을 보여 준 것이므로, 템플릿 엔진을 사용한다 해서 모든 상황에서 무조건 좋은 결과를 가져온다는 것은 아니다.

다만, 컨트롤 로직과 DOM관련 로직을 최대한 분리 시키고 또한 이런 툴들을 적극 활용하면 생각보다 어지러워질 수 있는 Backbone.js 특유의 컨트롤러를 좀 더 간결하게 그리고 MVC 패턴에 가깝게 유지할 수 있고, 따라서 그에 대한 이득이 분명히 생긴다는 점이 큰 의미를 가진다고 생각한다.
