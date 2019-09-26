# Simple Pagination (Vue.js)

This is just a simple practice for me.

Pagination is a component we ofen used. And this is based on Vue.js

**[DEMO Page](/https://hao830103.github.io/SimplePagination)**

![](https://i.imgur.com/fLKvsVs.png)

## How to use?

Use it like any Vue component, here is my demo code:

```javascript=
<pagination
    :MaxPage="12"
    :ShowenPages="3"
    @ChangePage="HandleEmitChangePage"
/>
```

#### MaxPage

```javascript=
:MaxPage="12"
```

Nothing special, it's the limit of this pagination.

#### ShowenPages

```javascript=
:ShowenPages="3"
```

This is set that how many pages you want to show in the same time.
**Basically, it will be nice to use odd number.**

#### Emit Function:

```javascript=
@ChangePage="HandleEmitChangePage"
```

ChangePage is the \$emit function name of this pagination, if you want to get the page numbers or do something while page change, you should do it with V-on.

And here is the emit function do:

```javascript=
this.$emit('ChangePage', pageNum);
```

## Custom Style

This is a big reason why I made this pagination for myself.

There is alot of project that need to use this component, so there will be alot of different style and CSS.

First, you need to know basic struct of this component.

```html=
<!-- this is the outer container div -->
<div>
  <ul>
    <!-- each page button is in <li> -->
    <li>
      <!-- page number could be other content -->
      <a>PageNum</a>
    </li>
  </ul>
</div>
```

You could use props to custom your style with your css.

```javascript=
props: {
    MaxPage: { type: Number },
    ShowenPages: { type: Number },
    ActiveClass: { default: 'active', type: String },
    PrevPageBtnHtml: { default: 'Prev', type: String },
    NextPageBtnHtml: { default: 'Next', type: String },
    PrevPageBtnClass: { default: '', type: String },
    NextPageBtnClass: { default: '', type: String },
    OuterDivClass: { default: 'pagination', type: String },
    UlClass: { default: '', type: String },
    LiClass: { default: '', type: String },
    FirsLastButton: { default: false, type: Boolean },
    FirstPageBtnClass: { default: '', type: String },
    LastPageBtnClass: { default: '', type: String },
    FirstPageBtnHtml: { default: '<<', type: String },
    LastPageBtnHtml: { default: '>>', type: String },
    OmittedPageClass: { default: 'dusabled', type: String },
    OmittedPageHtml: { default: '...', type: String }
}
```

It's very simple. Just put your class name into the props.

| Name              | Description                                                               | Type   |
| ----------------- | ------------------------------------------------------------------------- | ------ |
| MaxPage           | Set the limit page number                                                 | number |
| ShowenPages       | Set how may pages you want to display                                     | number |
| ActiveClass       | The style while you click and active page                                 | string |
| PrevPageBtnHtml   | Previous page button html, Default is 'Prev'                              | string |
| NextPageBtnHtml   | Next Page button html, Default is 'Next'                                  | string |
| PrevPageBtnClass  | Previous page button's class name                                         | string |
| NextPageBtnClass  | Next page button's class name                                             | string |
| OuterDivClass     | Container div's class name                                                | string |
| UlClass           | Ul's class name                                                           | string |
| LiClass           | Each li will use this class name                                          | string |
| FirsLastButton    | Show the First and Last page button, but it will make Omitted sign hidden | bool   |
| FirstPageBtnClass | First page button's class name                                            | string |
| LastPageBtnClass  | Last page button's class name                                             | string |
| OmittedPageClass  | Omitted sign's class name                                                 | string |
| OmittedPageHtml   | Omitted sign's html, default is '...'                                     | string |

That's it.
