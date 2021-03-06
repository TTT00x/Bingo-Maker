<template lang="pug">
.UrlViewer-Wrapper
  p.UrlViewer-Description
    |{{ description }}
    br
    span (現在のURLでは内容を保持できません。下記のURLを共有してください。)
  textarea.UrlViewer#urlViewer(@focus="select", v-model="nextUrl", readonly)
  .UrlViewer-BtnWrapper
    button.UrlViewer-Btn_Copy(@click="copy") コピーする
    a.UrlViewer-Btn_Tweet(:href="tweetHref", rel="nofollow", target="_blank") ツイートする
  .UrlViewer-BtnWrapper(v-if="mode === 'make'")
    a.UrlViewer-Btn_Play(:href="nextUrl", target="_blank") このビンゴで遊ぶ
  .UrlViewer-Notification(v-if="copyComplete")
    p コピーしました
</template>
<script>
  export default {
    props: ['title', 'mode', 'size', 'itemBg', 'itemCont'],
    data() {
      return {
        domain: document.domain === 'localhost' ? `http://localhost:8080` : `http://${document.domain}/bingo-maker/`,
        copyComplete: false
      }
    },
    methods: {
      select: (e) => {
        e.currentTarget.select();
      },
      copy: function () {
        const textArea = document.getElementById('urlViewer');
        textArea.select();
        document.execCommand('copy');
        this.copyComplete = true;
        setTimeout(() => {
          this.copyComplete = false;
        }, 3000);
      }
    },
    computed: {
      modeIsMake: function () {
        return this.mode === 'make';
      },
      modeIsPlay: function () {
        return this.mode === 'play';
      },
      modeIsResult: function () {
        return this.mode === 'result';
      },
      description: function () {
        if (this.modeIsMake) {
          return 'ビンゴを共有する'
        } else if (this.modeIsPlay) {
          return '結果を共有する'
        }
      },
      nextUrl: function() {
        if (!this.domain) return;
        return `${this.domain}/?title=${this.encodeTitle}&mode=${this.nextMode}&size=${this.size}${this.encodeContents}`;
      },
      nextMode: function () {
        if (this.modeIsMake) {
          return 'play';
        } else if (this.modeIsPlay) {
          return 'result';
        } else if (this.modeIsResult) {
          return this.mode;
        }
      },
      encodeTitle: function () {
        return encodeURIComponent(this.title);
      },
      encodeContents: function () {
        const cont = this.itemCont;
        const len = cont.length;
        let data = '';
        let datum;
        let openParam;
        for (let i = 0; i < len; i += 1) {
          datum = encodeURIComponent(cont[i].data);
          if (datum) {
            if (cont[i].open) {
              openParam = `&open${i}=1`;
            } else {
              openParam = '';
            }
            data = `${data}&cont${i}=${datum}${openParam}`
          }
        }
        return data;
      },
      tweetHref: function () {
        let description;
        if (this.modeIsMake) {
          description = 'ビンゴを作りました！';
        } else if (this.modeIsPlay) {
          description = 'ビンゴで遊びました！'
        }
        description = encodeURI(`${description} | `);
        const text = description + this.encodeTitle;
        return `http://twitter.com/share?text=${text}&url=${this.nextUrl}`
      }
    }
  };
</script>
<style scoped>
$bingo-border: #2c3e50;
$hilight: #f5bebe;

.UrlViewer-Wrapper {
  margin: 48px auto 0;
  max-width: 500px;
  width: 100%;
}

.UrlViewer-Description {
  font-size: 14px;
  text-align: center;
  line-height: 140%;

  span {
    color: #5b5b5b;
    font-size: 12px;
  }
}

.UrlViewer {
  border: 1px solid $bingo-border;
  font-size: 14px;
  line-height: 160%;
  margin-top: 8px;
  padding: 10px;
  resize: none;
  text-align: left;
  width: 100%;
  word-break: break-all;

  &:focus {
    background-color: #fafafa;
    outline: none;
  }
}

.UrlViewer-BtnWrapper {
  display: flex;
  justify-content: center;
  margin-top: 16px;
}

.UrlViewer-Btn_Copy,
.UrlViewer-Btn_Tweet,
.UrlViewer-Btn_Play {
  display: block;
  font-size: 14px;
  padding: 5px 10px;
  text-decoration: none;

  &:hover {
    cursor: pointer;
  }
}

.UrlViewer-Btn_Copy {
  background-color: #fff;
  border: 1px solid $bingo-border;
  color: $bingo-border;
  margin-right: 16px;

  &:hover {
    background-color: $bingo-border;
    color: #fff;
  }
}

.UrlViewer-Btn_Tweet {
  background-color: $bingo-border;
  border: 1px solid $bingo-border;
  color: #fff;

  &:hover {
    background-color: #fff;
    color: $bingo-border;
  }
}

.UrlViewer-Btn_Play {
  background-color: #fff;
  border: 1px solid $bingo-border;
  color: $bingo-border;

  &:hover {
    background-color: $bingo-border;
    color: #fff;
  }
}

.UrlViewer-Notification {
  background-color: $bingo-border;
  border-radius: 4px;
  color: #fff;
  display: flex;
  align-items: center;
  justify-content: center;
  position: fixed;
  top: calc(50% - 25px);
  left: calc(50% - 95px);
  font-size: 14px;
  height: 50px;
  padding: 10px;
  text-align: center;
  width: 190px;
}
</style>
