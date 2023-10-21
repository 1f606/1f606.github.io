<template>
  <div class="container">
    <div ref="$wrap" :style="{paddingTop: `${paddingTop}px`, paddingBottom: `${paddingBottom}px`, position: 'relative'}">
      <div
        :id="index === 0 ? 'top' : (index === (NODENUM - 1) ? 'bottom' : '')"
        v-for="(item, index) in renderList"
        :key="item.item"
        :ref="index === 0 ? '$topElement' : (index === 5 ? '$upAnchor' : (index === 10 ? '$downAnchor' : index === NODENUM - 1 ? '$bottomElement' : null))"
      >
        <div :style="{height: item.height}">
          {{item.item}}
        </div>
      </div>
    </div>
  </div>
</template>

<script>
const arr = [];
for (let i = 0; i < 20000; i++) {
  arr.push({
    item: i,
    height: i % 4 === 0 ?  '110px' : '240px'
  });
}
let i = 1;
const NODENUM = 20;

// eslint-disable-next-line no-unused-vars
function throttle (fn, ms) {
  let timer;
  return function() {
    const ctx = this;
    const args = arguments;
    clearTimeout(timer);
    timer = setTimeout(function () {
      fn.apply(ctx, args);
    }, ms);
  };
}
export default {
  name: 'VirtualScroll',
  data () {
    return {
      start: 0,
      end: NODENUM,
      paddingTop: 0,
      paddingBottom: 0,
      observer: null,
      currentArr: arr.slice(0, 40),
      NODENUM: 20,
      config: {
        isRequesting: false,
        paddingTopArr: [],
        paddingBottomArr: [],
        preScrollTop: 0,
        syncStart: 0,
        setting: false,
        maxScrollHeight: 0
      }
    };
  },
  computed: {
    renderList () {
      return this.currentArr.slice(this.start, this.end);
    }
  },
  watch: {
    renderList: {
      handler () {
        this.resetObservation();
        this.$nextTick(() => {
          this.initiateScrollObserver();
          this.config.maxScrollHeight = Math.max(this.config.maxScrollHeight, this.$refs.$wrap.scrollHeight);
        });
      },
      immediate: true
    }
  },
  mounted () {
    this.scrollEventListner = throttle(() => {
      const scrollTop = document.getElementsByClassName('container')[0].scrollTop;
      let index = this.config.paddingTopArr.findIndex(e => e > scrollTop);
      index = index <= 0 ? 0 : index;
      const len = this.config.paddingTopArr.length;
      len && (this.config.paddingTopArr[len - 1] < scrollTop) && (index = len);
      const newStart = index * 10;
      const newEnd = index * 10 + NODENUM;
      if (newStart === this.config.syncStart) {
        this.config.preScrollTop = scrollTop;
        return;
      }
      this.updateState(newStart, newEnd, scrollTop > this.config.preScrollTop, true);
      this.config.preScrollTop = scrollTop;
    }, 100);
    document.getElementsByClassName('container')[0].addEventListener('scroll', this.scrollEventListner);
  },
  destroyed () {
    document.getElementsByClassName('container')[0].removeEventListener('scroll', this.scrollEventListner);
  },
  methods: {
    setStart (v) {
      this.start = v;
    },
    setEnd (v) {
      this.end = v;
    },
    setPaddingTop (v) {
      this.paddingTop = v;
    },
    setPaddingBottom (v) {
      this.paddingBottom = v;
    },
    setObserver (v) {
      this.observer = v;
    },
    resetObservation () {
      this.observer && this.observer.unobserve(this.$refs.$bottomElement?.[0]);
      this.observer && this.observer.unobserve(this.$refs.$topElement?.[0]);
    },
    initiateScrollObserver () {
      const options = {
        root: null,
        rootMargin: '0px',
        threshold: 0.1
      };
      const Observer = new IntersectionObserver((entries) => {
        entries.forEach(entry => {
          const listLength = this.currentArr.length;
          // 向下滚动
          if (entry.isIntersecting && entry.target.id === 'bottom') {
            const maxStartIndex = listLength - 1 - NODENUM;
            const maxEndIndex = listLength - 1;
            const newStart = (this.end - 10) <= maxStartIndex ? this.end - 10 : maxStartIndex;
            const newEnd = (this.end + 10) <= maxEndIndex ? this.end + 10 : maxEndIndex;
            if (newEnd + 10 >= maxEndIndex && !this.config.isRequesting) {
              this.currentArr.push(...arr.slice(i * 40, (i + 1) * 40));
              i++;
            }
            // 似乎用不上
            // if (this.end + 10 > maxEndIndex) return;
            this.updateState(newStart, newEnd, true);
          }
          // 向上滚动
          if (entry.isIntersecting && entry.target.id === 'top') {
            const newEnd = this.end === NODENUM ? NODENUM : (this.end - 10 > NODENUM ? this.end - 10 : NODENUM);
            const newStart = this.start === 0 ? 0 : (this.start - 10 > 0 ? this.start - 10 : 0);
            this.updateState(newStart, newEnd, false);
          }
        });
      }, options);
      if (this.$refs.$topElement?.[0]) {
        Observer.observe(this.$refs.$topElement?.[0]);
      }
      if (this.$refs.$bottomElement?.[0]) {
        Observer.observe(this.$refs.$bottomElement?.[0]);
      }
      this.setObserver(Observer);
    },
    updateState (newStart, newEnd, isDown) {
      if (this.config.setting) return;
      this.config.syncStart = newStart;
      if (this.start !== newStart || this.end !== newEnd) {
        this.config.setting = true;
        this.setStart(newStart);
        this.setEnd(newEnd);
        const page = Math.floor((newStart / 10)) - 1;
        if (isDown) {
          newStart !== 0 && !this.config.paddingTopArr[page] && (this.config.paddingTopArr[page] = this.$refs.$downAnchor?.[0].offsetTop);
          this.setPaddingTop(this.config.paddingTopArr[page]);
          this.setPaddingBottom(this.config.paddingBottomArr[page] || 0);
        } else {
          let newPaddingBottom = this.$refs.$wrap.scrollHeight - this.$refs.$downAnchor?.[0].offsetTop;
          if (newStart === 0) {
            newPaddingBottom = this.config.maxScrollHeight - this.config.paddingTopArr[1];
          } else {
            this.config.paddingBottomArr[page] = newPaddingBottom;
          }
          this.setPaddingTop(this.config.paddingTopArr[page] || 0);
          this.setPaddingBottom(newPaddingBottom);
        }
        setTimeout(() => { this.config.setting = false; }, 0);
      }
    }
  }
};
</script>

<style scoped>
.item {
  width: 100%;
  border-bottom: 1px solid #eee;
}
.container {
  /*width: 30vw;*/
  height: 30vh;
  overflow: scroll;
}
</style>
