<template>
  <div class="contents">
    <h1 class="pg-heading">TOC</h1>
    <div>
      <div>
        <textarea v-model="value" placeholder="ここにテキストを入力してください" class="input-area" />
      </div>
      <h2 class="pg-title">preview</h2>
      <article class="p-article" v-html="convertedValue" />
      <h2 class="pg-title">code</h2>
      <CodeHighlight lang="html hljs xml" :html="convertedValue" />
    </div>
  </div>
</template>
<script>
const head = {
  title: 'Playground',
  description: 'playground',
  keywords: ['playground'],
  breadcrumbs: [],
};

export default {
  data() {
    return {
      value: '',
    };
  },
  computed: {
    convertedValue() {
      if (!this.value) return this.value;
      const array = this.value.trim().split(/\r\n|\n/g);

      // toc作成
      let toc = '';
      let heading = '';
      let h2 = 1;
      let h3 = 1;
      for (let i = 0, len = array.length; i < len; i += 1) {
        let id = '';
        if (this.isPrefixMatch('H2.', array[i])) {
          id = `#h2-${h2}`;
          h2 += 1;
        } else {
          id = `#h3-${h3}`;
          h3 += 1;
        }
        const text = this.getHeadingText(array[i]);
        toc += `    <li><a href='${id}'>${text}</a>`;
        if (this.isPrefixMatch('H3.', array[i + 1])) {
          if (this.isPrefixMatch('H3.', array[i])) {
            toc += '</li>\n    ';
          } else {
            toc += '\n      <ul>\n    ';
          }
        } else if (this.isPrefixMatch('H3.', array[i])) {
          toc += '</li>\n      </ul>\n    </li>\n';
        } else {
          toc += '</li>\n';
        }

        // headingを作成
        heading += this.createHeading(id, text);
      }

      return `<!-- toc -->
              <nav class='toc'>
                <ul>
              ${toc}
                </ul>
              </nav>

              <!-- heading -->
              ${heading}
              `;
    },
  },
  methods: {
    isPrefixMatch(type, value) {
      const regexp = new RegExp(`^${type}`);
      return regexp.test(value);
    },
    getHeadingText(text) {
      return text
        .trim()
        .replace(/H2.|H3.|[\s|\t]+\d*$/g, '')
        .trim();
    },
    createHeading(id, text) {
      const headingId = id.slice(1);
      if (/^h2-/.test(headingId)) {
        return `<h2 id='${headingId}'>${text}</h2>\n`;
      }
      if (/^h3-/.test(headingId)) {
        return `<h3 id='${headingId}'>${text}</h3>\n`;
      }
    },
  },
};
</script>
<style>
.contents {
  width: 500px;
}

.input-area {
  width: 100%;
  height: 200px;
  resize: none;
}
</style>
