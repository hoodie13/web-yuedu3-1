<template>
  <div
    class="chapter-wrapper"
    :style="bodyTheme"
    :class="{ night: isNight, day: !isNight }"
  >
    <div class="tool-bar" :style="leftBarTheme">
      <div class="tools">
        <el-popover
          placement="right"
          :width="cataWidth"
          trigger="click"
          :visible-arrow="false"
          v-model="popCataVisible"
          popper-class="pop-cata"
        >
          <PopCata @getContent="getContent" ref="popCata" class="popup" />

          <div
            class="tool-icon"
            :class="{ 'no-point': noPoint }"
            slot="reference"
          >
            <div class="iconfont">
              &#58905;
            </div>
            <div class="icon-text">目录</div>
          </div>
        </el-popover>
        <el-popover
          placement="bottom-right"
          width="470"
          trigger="click"
          :visible-arrow="false"
          v-model="readSettingsVisible"
          popper-class="pop-setting"
        >
          <ReadSettings class="popup" />

          <div
            class="tool-icon"
            :class="{ 'no-point': noPoint }"
            slot="reference"
          >
            <div class="iconfont">
              &#58971;
            </div>
            <div class="icon-text">设置</div>
          </div>
        </el-popover>
        <div class="tool-icon" @click="toShelf">
          <div class="iconfont">
            &#58892;
          </div>
          <div class="icon-text">书架</div>
        </div>
        <div
          class="tool-icon"
          :class="{ 'no-point': noPoint }"
          @click="saveRecord"
        >
          <div class="iconfont">
            &#58957;
          </div>
          <div class="icon-text">保存</div>
        </div>
        <div class="tool-icon" :class="{ 'no-point': noPoint }" @click="toNew">
          <div class="iconfont">
            &#58890;
          </div>
          <div class="icon-text">同步</div>
        </div>
        <div class="tool-icon" :class="{ 'no-point': noPoint }" @click="toTop">
          <div class="iconfont">
            &#58914;
          </div>
          <div class="icon-text">顶部</div>
        </div>
        <div
          class="tool-icon"
          :class="{ 'no-point': noPoint }"
          @click="toBottom"
        >
          <div class="iconfont">
            &#58915;
          </div>
          <div class="icon-text">底部</div>
        </div>
      </div>
    </div>
    <div class="read-bar" :style="rightBarTheme">
      <div class="tools">
        <div
          class="tool-icon"
          :class="{ 'no-point': noPoint }"
          @click="toLastChapter"
        >
          <div class="iconfont">
            &#58920;
          </div>
        </div>
        <div
          class="tool-icon"
          :class="{ 'no-point': noPoint }"
          @click="toNextChapter"
        >
          <div class="iconfont">
            &#58913;
          </div>
        </div>
      </div>
    </div>
    <div class="chapter-bar"></div>
    <div class="chapter" ref="content" :style="chapterTheme">
      <div class="content">
        <div class="top-bar" ref="top"></div>
        <div class="title" ref="title" v-if="show">{{ title }}</div>
        <Pcontent :carray="content" />
        <div class="bottom-bar" ref="bottom"></div>
      </div>
    </div>
  </div>
</template>

<script>
import PopCata from "../components/PopCatalog.vue";
import ReadSettings from "../components/ReadSettings.vue";
import Pcontent from "../components/Content.vue";
import Axios from "axios";
import jump from "../plugins/jump";
import config from "../plugins/config";
export default {
  components: {
    PopCata,
    Pcontent,
    ReadSettings
  },
  created() {
    var config = JSON.parse(localStorage.getItem("config"));
    if (config != null) this.$store.commit("setConfig", config);
  },
  beforeCreate() {
    let config = JSON.parse(localStorage.getItem("config"));
    if (config != null) this.$store.commit("setConfig", config);
  },
  mounted() {
    this.loading = this.$loading({
      target: this.$refs.content,
      lock: true,
      text: "正在获取内容",
      spinner: "el-icon-loading",
      background: "rgba(0,0,0,0)"
    });
    //获取书籍数据
    const that = this;
    let bookUrl = sessionStorage.getItem("bookUrl");
    var book = JSON.parse(localStorage.getItem(bookUrl));
    this.getBook(bookUrl).then(
      res => {
        let newBook = res.data.data;
        let bookName = newBook.name;
        var chapterIndex = 0;
        var chapterPos = 0;
        if (newBook.webDurChapterTime > newBook.durChapterTime) {
          chapterIndex = newBook.webChapterIndex;
          chapterPos = newBook.webChapterPos;
        } else {
          chapterIndex = newBook.durChapterIndex;
          chapterPos = newBook.durChapterPos;
        }
        window.console.log(
          "getBook webDurChapterTime is " +
            newBook.webDurChapterTime +
            " durChapterTime is " +
            newBook.durChapterTime +
            " chapterIndex is " +
            chapterIndex +
            " chapterPos is " +
            chapterPos
        );
        book = {
          bookName: bookName,
          bookUrl: bookUrl,
          index: chapterIndex,
          chapterPos: chapterPos
        };
        localStorage.setItem(bookUrl, JSON.stringify(book));
        this.getCatalog(bookUrl);
        window.addEventListener("scroll", this.handleScroll);
        window.addEventListener("keyup", function(event) {
          window.console.log("Event key is " + event.key);
          let t = new Date().getTime();
          switch (event.key) {
            case "ArrowLeft":
              event.stopPropagation();
              event.preventDefault();
              that.toLastChapter();
              break;
            case "ArrowRight":
              event.stopPropagation();
              event.preventDefault();
              that.toNextChapter();
              break;
            case "ArrowUp":
              if (t - this.oldT2 > 500) {
                event.stopPropagation();
                event.preventDefault();
                if (document.documentElement.scrollTop === 0) {
                  that.$message.warning("已到达页面顶部");
                } else {
                  jump(0 - document.documentElement.clientHeight + 200);
                }
                this.oldT2 = t;
              }
              break;
            case "ArrowDown":
              if (t - this.oldT2 > 500) {
                event.stopPropagation();
                event.preventDefault();
                if (
                  document.documentElement.clientHeight +
                    document.documentElement.scrollTop ===
                  document.documentElement.scrollHeight
                ) {
                  that.$message.warning("已到达页面底部");
                } else {
                  jump(document.documentElement.clientHeight - 200);
                }
                this.oldT2 = t;
              }
              break;
            case "s":
              if (t - this.oldT2 > 2000) {
                that.saveRecord(true);
                this.oldT2 = t;
              }
              break;
            case "n":
              if (t - this.oldT2 > 2000) {
                that.toNew();
                this.oldT2 = t;
              }
              break;
            case "m":
              if (t - this.oldT2 > 2000) {
                that.setBookmark();
                this.oldT2 = t;
              }
              break;
          }
        });
      },
      err => {
        that.loading.close();
        that.$message.error("获取书籍失败");
        throw err;
      }
    );
  },
  watch: {
    chapterName(to) {
      this.title = to;
    },
    content() {
      this.$store.commit("setContentLoading", false);
    },
    theme(theme) {
      if (theme == 6) {
        this.isNight = true;
      } else {
        this.isNight = false;
      }
    },
    bodyColor(color) {
      this.bodyTheme.background = color;
    },
    chapterColor(color) {
      this.chapterTheme.background = color;
    },
    readWidth(width) {
      this.chapterTheme.width = width;
      let leftToolMargin = -((parseInt(width) + 130) / 2 + 68) + "px";
      let rightToolMargin = -((parseInt(width) + 130) / 2 + 52) + "px";
      this.leftBarTheme.marginLeft = leftToolMargin;
      this.rightBarTheme.marginRight = rightToolMargin;
    },
    popupColor(color) {
      this.leftBarTheme.background = color;
      this.rightBarTheme.background = color;
    },
    readSettingsVisible(visible) {
      if (!visible) {
        let configText = JSON.stringify(this.$store.state.config);
        localStorage.setItem("config", configText);
      }
    }
  },
  data() {
    return {
      title: "",
      content: [],
      noPoint: true,
      isNight: this.$store.state.config.theme == 6,
      oldY: 0,
      oldT: new Date().getTime(),
      oldT2: new Date().getTime(),
      bodyTheme: {
        background: config.themes[this.$store.state.config.theme].body
      },
      chapterTheme: {
        background: config.themes[this.$store.state.config.theme].content,
        width: this.$store.state.config.readWidth - 130 + "px"
      },
      leftBarTheme: {
        background: config.themes[this.$store.state.config.theme].popup,
        marginLeft: -(this.$store.state.config.readWidth / 2 + 68) + "px"
      },
      rightBarTheme: {
        background: config.themes[this.$store.state.config.theme].popup,
        marginRight: -(this.$store.state.config.readWidth / 2 + 52) + "px"
      }
    };
  },
  computed: {
    catalog() {
      return this.$store.state.catalog;
    },
    windowHeight() {
      return window.innerHeight;
    },
    contentHeight() {
      return this.$refs.content.offsetHeight;
    },
    popCataVisible: {
      get() {
        return this.$store.state.popCataVisible;
      },
      set(value) {
        this.$store.commit("setPopCataVisible", value);
      }
    },
    readSettingsVisible: {
      get() {
        return this.$store.state.readSettingsVisible;
      },
      set(value) {
        this.$store.commit("setReadSettingsVisible", value);
      }
    },
    config() {
      return this.$store.state.config;
    },
    theme() {
      return this.config.theme;
    },
    bodyColor() {
      return config.themes[this.config.theme].body;
    },
    chapterColor() {
      return config.themes[this.config.theme].content;
    },
    popupColor() {
      return config.themes[this.config.theme].popup;
    },
    readWidth() {
      return this.$store.state.config.readWidth - 130 + "px";
    },
    cataWidth() {
      return this.$store.state.config.readWidth - 33;
    },
    show() {
      return this.$store.state.showContent;
    }
  },
  methods: {
    handleScroll() {
      let scrollY = window.scrollY;
      let t = new Date().getTime();
      //阅读超过1分钟保存一次阅读进度
      if (scrollY - this.oldY > 1000 && t - this.oldT > 60000) {
        this.oldT = t;
        this.oldY = scrollY;
        this.saveRecord(false);
      }
    },
    saveRecord(showInfo) {
      let totalH =
        document.body.scrollHeight || document.documentElement.scrollHeight;
      let clientH = window.innerHeight || document.documentElement.clientHeight;
      let validH = totalH - clientH;
      let scrollH =
        document.body.scrollTop || document.documentElement.scrollTop;
      let index = this.$store.state.readingBook.index;
      let len = this.$store.state.readingBook.catalog[index].len;
      let chapterIndex = this.$store.state.readingBook.catalog[index].index;
      let pos = ((len * scrollH) / validH).toFixed(0);
      let bookUrl = sessionStorage.getItem("bookUrl");
      if (pos == null) pos = 0;
      if (chapterIndex != null) {
        window.console.log(
          "saveRecord chapterIndex is " + chapterIndex + " pos is " + pos
        );
        Axios.get(
          "/saveReadRecord?url=" +
            encodeURIComponent(bookUrl) +
            "&chapterIndex=" +
            chapterIndex +
            "&pos=" +
            pos
        );
        this.$store.state.readingBook.chapterIndex = chapterIndex;
        this.$store.state.readingBook.chapterPos = pos;
        this.oldT = new Date().getTime();
        this.oldY = window.scrollY;
        if (showInfo) {
          this.$message.info(
            "正在保存阅读进度到本地，进度索引为[" + index + "." + pos + "]。"
          );
        }
      }
    },
    getBook(bookUrl) {
      return Axios.get("/getBook?url=" + encodeURIComponent(bookUrl));
    },
    getCatalog(bookUrl) {
      let that = this;
      Axios.get("/getChapterList?url=" + encodeURIComponent(bookUrl)).then(
        res => {
          let catalog = res.data.data;
          let book = JSON.parse(localStorage.getItem(bookUrl));
          book.catalog = catalog;
          that.$store.commit("setReadingBook", book);
          var index = that.$store.state.readingBook.index || 0;
          this.getContent(index);
        },
        err => {
          that.loading.close();
          that.$message.error("获取书籍目录失败");
          throw err;
        }
      );
    },
    getContent(index) {
      //展示进度条
      this.$store.commit("setShowContent", false);
      if (!this.loading.visible) {
        this.loading = this.$loading({
          target: this.$refs.content,
          lock: true,
          text: "正在获取内容",
          spinner: "el-icon-loading",
          background: "rgba(0,0,0,0)"
        });
      }
      //保存阅读进度
      let bookUrl = sessionStorage.getItem("bookUrl");
      let book = JSON.parse(localStorage.getItem(bookUrl));
      book.index = index;
      localStorage.setItem(bookUrl, JSON.stringify(book));
      this.$store.state.readingBook.index = index;
      //let chapterUrl = this.$store.state.readingBook.catalog[index].url;
      let chapterName = this.$store.state.readingBook.catalog[index].title;
      let chapterIndex = this.$store.state.readingBook.catalog[index].index;
      this.title = chapterName;
      //强制滚回顶层
      jump(this.$refs.top, { duration: 0 });
      let that = this;
      Axios.get(
        "/getBookContent?url=" +
          encodeURIComponent(bookUrl) +
          "&index=" +
          chapterIndex
      ).then(
        res => {
          let data = res.data.data;
          this.$store.state.readingBook.catalog[index].len = data.length;
          let dataArray = data.split("\n\n");
          let contentData = "";
          if (dataArray.length > 1) {
            contentData = dataArray[1].split("\n");
          } else {
            contentData = dataArray[0].split("\n");
          }
          that.content = contentData;
          this.$store.commit("setContentLoading", true);
          that.loading.close();
          that.noPoint = false;
          that.$store.commit("setShowContent", true);
          let pos = this.$store.state.readingBook.chapterPos;
          if (pos > 100) {
            this.$message.info(
              "当前章最新进度索引为[" +
                index +
                "." +
                pos +
                "]，请点击同步跳转阅读"
            );
          }
          this.oldT = new Date().getTime();
          this.oldY = window.scrollY;
        },
        err => {
          that.$message.error("获取章节内容失败");
          that.content = "　　获取章节内容失败！";
          throw err;
        }
      );
    },
    toNew() {
      let totalH =
        document.body.scrollHeight || document.documentElement.scrollHeight;
      let clientH = window.innerHeight || document.documentElement.clientHeight;
      let validH = totalH - clientH;
      let index = this.$store.state.readingBook.index;
      let len = this.$store.state.readingBook.catalog[index].len;
      let chapterPos = this.$store.state.readingBook.chapterPos;
      let scrollH = (validH * chapterPos) / len;
      document.documentElement.scrollTop = scrollH;
      /*
      this.$message.info(
        "同步阅读进度到最近处，进度索引为[" + index + "." + chapterPos + "]。"
      );*/
    },
    setBookmark() {
      let that = this;
      let index = this.$store.state.readingBook.index;
      let chapterIndex = this.$store.state.readingBook.catalog[index].index;
      let title = this.$store.state.readingBook.catalog[index].title;
      let bookUrl = sessionStorage.getItem("bookUrl");
      let bookText = window
        .getSelection()
        .toString()
        .trim();
      if (bookText != "") {
        let pos = title.length + that.content.join("\n").indexOf(bookText);
        window.console.log(
          "setBookmark chapterIndex is " +
            chapterIndex +
            " chapterPos is " +
            pos
        );
        Axios.get(
          "/setBookmark?url=" +
            encodeURIComponent(bookUrl) +
            "&chapterIndex=" +
            chapterIndex +
            "&pos=" +
            pos +
            "&bookText=" +
            bookText
        );
        this.$store.state.readingBook.chapterIndex = chapterIndex;
        this.$store.state.readingBook.chapterPos = pos;
        this.oldT = new Date().getTime();
        this.oldY = window.scrollY;
        this.$message.info("正在保存书签");
      }
    },
    toTop() {
      jump(this.$refs.top);
    },
    toBottom() {
      jump(this.$refs.bottom);
    },
    toNextChapter() {
      this.$store.commit("setContentLoading", true);
      let index = this.$store.state.readingBook.index;
      index++;
      if (typeof this.$store.state.readingBook.catalog[index] !== "undefined") {
        this.$store.state.readingBook.chapterPos = 0;
        this.getContent(index);
      } else {
        this.$message.error("本章是最后一章");
      }
    },
    toLastChapter() {
      this.$store.commit("setContentLoading", true);
      let index = this.$store.state.readingBook.index;
      index--;
      if (typeof this.$store.state.readingBook.catalog[index] !== "undefined") {
        this.$store.state.readingBook.chapterPos = 0;
        this.getContent(index);
      } else {
        this.$message.error("本章是第一章");
      }
    },
    toShelf() {
      this.$router.push("/");
      this.saveRecord(false);
    }
  }
};
</script>

<style lang="stylus" scoped>
>>>.pop-setting {
  margin-left: 68px;
  top: 0;
}

>>>.pop-cata {
  margin-left: 10px;
}

.chapter-wrapper {
  padding: 0 4%;
  flex-direction: column;
  align-items: center;

  >>>.no-point {
    pointer-events: none;
  }

  .tool-bar {
    position: fixed;
    top: 0;
    left: 50%;
    z-index: 100;

    .tools {
      display: flex;
      flex-direction: column;

      .tool-icon {
        font-size: 18px;
        width: 58px;
        height: 48px;
        text-align: center;
        padding-top: 12px;
        cursor: pointer;
        outline: none;

        .iconfont {
          font-family: iconfont;
          width: 16px;
          height: 16px;
          font-size: 16px;
          margin: 0 auto 6px;
        }

        .icon-text {
          font-size: 12px;
        }
      }
    }
  }

  .read-bar {
    position: fixed;
    bottom: 0;
    right: 50%;
    z-index: 100;

    .tools {
      display: flex;
      flex-direction: column;

      .tool-icon {
        font-size: 18px;
        width: 42px;
        height: 31px;
        padding-top: 12px;
        text-align: center;
        align-items: center;
        cursor: pointer;
        outline: none;
        margin-top: -1px;

        .iconfont {
          font-family: iconfont;
          width: 16px;
          height: 16px;
          font-size: 16px;
          margin: 0 auto 6px;
        }
      }
    }
  }

  .chapter-bar {
    .el-breadcrumb {
      .item {
        font-size: 14px;
        color: #606266;
      }
    }
  }

  .chapter {
    font-family: 'Microsoft YaHei', PingFangSC-Regular, HelveticaNeue-Light, 'Helvetica Neue Light', sans-serif;
    text-align: left;
    padding: 0 65px;
    min-height: 100vh;
    width: 670px;
    margin: 0 auto;

    >>>.el-icon-loading {
      font-size: 36px;
      color: #B5B5B5;
    }

    >>>.el-loading-text {
      font-weight: 500;
      color: #B5B5B5;
    }

    .content {
      font-size: 18px;
      line-height: 1.8;
      overflow: hidden;
      font-family: 'Microsoft YaHei', PingFangSC-Regular, HelveticaNeue-Light, 'Helvetica Neue Light', sans-serif;

      .title {
        margin-bottom: 57px;
        font: 24px / 32px PingFangSC-Regular, HelveticaNeue-Light, 'Helvetica Neue Light', 'Microsoft YaHei', sans-serif;
      }

      .bottom-bar, .top-bar {
        height: 64px;
      }
    }
  }
}

.day {
  >>>.popup {
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.12), 0 0 6px rgba(0, 0, 0, 0.04);
  }

  >>>.tool-icon {
    border: 1px solid rgba(0, 0, 0, 0.1);
    margin-top: -1px;
    color: #000;

    .icon-text {
      color: rgba(0, 0, 0, 0.4);
    }
  }

  >>>.chapter {
    border: 1px solid #d8d8d8;
    color: #262626;
  }
}

.night {
  >>>.popup {
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.48), 0 0 6px rgba(0, 0, 0, 0.16);
  }

  >>>.tool-icon {
    border: 1px solid #444;
    margin-top: -1px;
    color: #666;

    .icon-text {
      color: #666;
    }
  }

  >>>.chapter {
    border: 1px solid #444;
    color: #666;
  }

  >>>.popper__arrow {
    background: #666;
  }
}
</style>
