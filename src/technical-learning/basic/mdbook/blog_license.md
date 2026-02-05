# blog license: creativecommons

## 一、Reference

> [creative commons](https://creativecommons.org/)

> [How can I license my content in a blogpost?](https://opensource.stackexchange.com/questions/365/how-can-i-license-my-content-in-a-blogpost)

## 二、Get License HTML

![license](https://github.com/Kua-Fu/blog-book-images/blob/main/mdbook/license1.png?raw=true)

![license](https://github.com/Kua-Fu/blog-book-images/blob/main/mdbook/license2.png?raw=true)

![license](https://github.com/Kua-Fu/blog-book-images/blob/main/mdbook/license3.png?raw=true)

## 三、Modify mdBook template

### 3.1 Add HTML 

`./theme/index.hbs`

```html

<div id="bottom"> 
	<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a>
</div>   	 
```

### 3.2 Add CSS

`./theme/css/general.css`

```CSS

#bottom{
    display: flex;
    justify-content: space-between;
    align-items: center;
}


```

