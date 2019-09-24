<template>
  <div :class="OuterDivClass">
    <ul :class="UlClass">
      <li @click="FirstPage" v-if="FirsLastButton" :class="FirstPageBtnClass">
        <a v-html="FirstPageBtnHtml"></a>
      </li>
      <li @click="PrevPage" :class="PrevPageBtnClass">
        <a v-html="PrevPageBtnHtml"></a>
      </li>
      <li
        @click="FirstPage"
        v-if="
          CurrentPage >= HiddenPageBase.Min &&
            !FirsLastButton &&
            this.MaxPage > this.ShowenPages + 1
        "
      >
        <a>1</a>
      </li>
      <li
        :class="OmittedPageClass"
        v-show="
          CurrentPage > HiddenPageBase.Min &&
            !FirsLastButton &&
            this.MaxPage > this.ShowenPages + 1
        "
      >
        <a v-html="OmittedPageHtml"></a>
      </li>
      <li
        v-for="pageNum in PageRange"
        :key="pageNum"
        :class="[CurrentPage === pageNum ? ActiveClass : '', LiClass]"
        @click="ChangePage(pageNum)"
      >
        <a>
          <span>{{ pageNum }}</span>
        </a>
      </li>
      <li
        :class="OmittedPageClass"
        v-show="
          CurrentPage < HiddenPageBase.Max &&
            !FirsLastButton &&
            this.MaxPage > this.ShowenPages + 1
        "
      >
        <a v-html="OmittedPageHtml"></a>
      </li>
      <li
        @click="LastPage"
        v-if="
          CurrentPage <= HiddenPageBase.Max &&
            !FirsLastButton &&
            this.MaxPage > this.ShowenPages + 1
        "
      >
        <a>{{ MaxPage }}</a>
      </li>
      <li @click="NextPage" :class="NextPageBtnClass">
        <a v-html="NextPageBtnHtml"></a>
      </li>
      <li @click="LastPage" v-if="FirsLastButton" :class="LastPageBtnClass">
        <a v-html="LastPageBtnHtml"></a>
      </li>
    </ul>
  </div>
</template>
<script>
export default {
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
  },
  data: function () {
    return {
      PageRange: [],
      CurrentPage: 1,
      HiddenPageBase: {
        Max: 0,
        Min: 0
      }
    }
  },
  mounted: function () {
    this.LimitCompute()
    this.SetPageRange()
    this.ChangePage(1)
  },
  watch: {
    MaxPage: function () {
      this.LimitCompute()
      this.ChangePage(1)
    }
  },
  methods: {
    ChangePage: function (pageNum) {
      this.CurrentPage = pageNum
      this.SetPageRange()
      this.$emit('ChangePage', pageNum)
    },
    LimitCompute: function () {
      if (this.MaxPage > this.ShowenPages + 1) {
        const base = Math.ceil(this.ShowenPages / 2)
        this.HiddenPageBase.Min = base + 1
        this.HiddenPageBase.Max = this.MaxPage - base
      }
    },
    SetPageRange: function () {
      let result = []
      if (this.MaxPage > this.ShowenPages + 1) {
        if (this.CurrentPage < this.HiddenPageBase.Min) {
          for (let i = 1; i <= 1 + (this.ShowenPages - 1); i++) {
            result.push(i)
          }
        } else if (this.CurrentPage > this.HiddenPageBase.Max) {
          for (
            let i = this.MaxPage - (this.ShowenPages - 1);
            i <= this.MaxPage;
            i++
          ) {
            result.push(i)
          }
        } else {
          const base = Math.floor(this.ShowenPages / 2)
          for (let i = base; i >= 1; i--) {
            result.push(this.CurrentPage - i)
          }
          result.push(this.CurrentPage)
          for (let i = 1; i <= base; i++) {
            result.push(this.CurrentPage + i)
          }
        }
      } else {
        for (let i = 1; i <= this.MaxPage; i++) {
          result.push(i)
        }
      }
      this.PageRange = result
    },
    PrevPage: function () {
      if (this.CurrentPage !== 1) {
        this.ChangePage(this.CurrentPage - 1)
      }
    },
    NextPage: function () {
      if (this.CurrentPage !== this.MaxPage) {
        this.ChangePage(this.CurrentPage + 1)
      }
    },
    FirstPage: function () {
      if (this.CurrentPage !== 1) {
        this.ChangePage(1)
      }
    },
    LastPage: function () {
      if (this.CurrentPage !== this.MaxPage) {
        this.ChangePage(this.MaxPage)
      }
    }
  }
}
</script>
<style scoped>
a {
  color: #0088cc;
  text-decoration: none;
}
a:hover {
  color: #005580;
  text-decoration: underline;
}
ul {
  padding: 0;
}

.pagination {
  height: 36px;
  margin: 18px 0;
}
.pagination ul {
  display: inline-block;
  *zoom: 1;
  margin-left: 0;
  margin-bottom: 0;
  -webkit-border-radius: 3px;
  -moz-border-radius: 3px;
  border-radius: 3px;
  -webkit-box-shadow: 0 1px 2px rgba(0, 0, 0, 0.05);
  -moz-box-shadow: 0 1px 2px rgba(0, 0, 0, 0.05);
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.05);
}
.pagination li {
  display: inline;
  cursor: pointer;
}
.pagination a {
  float: left;
  width: 3rem;
  text-align: center;
  line-height: 34px;
  text-decoration: none;
  border: 1px solid #ddd;
  border-left-width: 0;
}
.pagination a:hover,
.pagination .active a {
  background-color: #f5f5f5;
}
.pagination .active a {
  color: #999999;
  cursor: default;
}
.pagination .disabled span,
.pagination .disabled a,
.pagination .disabled a:hover {
  color: #999999;
  background-color: transparent;
  cursor: default;
}
.pagination li:first-child a {
  border-left-width: 1px;
  -webkit-border-radius: 3px 0 0 3px;
  -moz-border-radius: 3px 0 0 3px;
  border-radius: 3px 0 0 3px;
}
.pagination li:last-child a {
  -webkit-border-radius: 0 3px 3px 0;
  -moz-border-radius: 0 3px 3px 0;
  border-radius: 0 3px 3px 0;
}
.pagination-centered {
  text-align: center;
}
.pagination-right {
  text-align: right;
}
</style>
