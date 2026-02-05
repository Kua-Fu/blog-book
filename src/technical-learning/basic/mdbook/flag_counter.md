# visitor analytics: flag counter

## 一、Reference

> [flag counter](https://flagcounter.com/)

## 二、Get Custom HTML

![flagcounter](https://github.com/Kua-Fu/blog-book-images/blob/main/mdbook/flagCounter1.png?raw=true)

![flagcounter](https://github.com/Kua-Fu/blog-book-images/blob/main/mdbook/flagCounter2.png?raw=true)

![flagcounter](https://github.com/Kua-Fu/blog-book-images/blob/main/mdbook/flagCounter3.png?raw=true)

## 三、Modify mdBook template

### 3.1 Add HTML 

`./theme/index.hbs`

```html

<div id="bottom"> 
	<a href="https://info.flagcounter.com/42Wy"><img src="https://s01.flagcounter.com/count/42Wy/bg_FFFFFF/txt_000000/border_CCCCCC/columns_3/maxflags_9/viewers_0/labels_0/pageviews_1/flags_0/percent_0/" alt="Flag Counter" border="0"></a>
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
