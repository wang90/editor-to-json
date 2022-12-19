<template>
  <div class="app">
    <div class="editor view">
      <div class="btns">
        <button>转换</button>
        <button @click="importEditor">倒入</button>
        <button @click="addNode">添加节点</button>
      </div>
      <div class="rich-text"
        ref="richTextRef"
        contenteditable="true"
        >
         
      </div>
    </div>
    <div class="preveiw view">
      <div class="btns">
        <button>转换</button>
        <button @click="importEditor">倒入</button>
        <button @click="addNode">添加节点</button>
      </div>
      <div>
        <div class="rich-text">
          <base-node v-for="(n) in nodes"
            :nodeid="n.nodeid"
            :values="n.values"
          ></base-node>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { reactive, ref, onMounted, onUnmounted } from "vue";
import importJson from "./json/data.json";
import BaseNode from './components/BaseNode.vue';

const json: any = reactive(importJson);
const nodes: any = reactive([]);

const richTextRef: any = ref(null);

const tree = {};

const importEditor = () => {
  let html = '';
  for (let i = 0; i < json.nodes.length; i++) {
    const { _id, values } = json.nodes[i];
    const __vals: any = [];
    let __html = '';

    for (let j = 0; j < values.length; j++) { 
      const __uuid = uuid();
      const __v = values[j];

      if ( __v.from === 'model') {
        __html += `<font source="${__v.from}" id="${__uuid}">${__v.value}</font>`;
        tree[__uuid] = __v.value;
      } else {
        __html += `<font source="${ __v.from }" id="${ __uuid }">${ __v.value }</font>`;
      }

      __vals.push({
        uuid: __uuid,
        value: __v.value,
        from: __v.from,
      })
    }

    nodes.push({
      nodeid: _id,
      id: _id,
      values: __vals,
    })
    html += `<span data-nodeid="${ _id }">${ __html }</span>`
  } 

  richTextRef.value.innerHTML = html;

};

const addNode = () => {
  const nodeid = Date.now();
  nodes.push({
    '_id': nodeid,
    'nodeid': nodeid,
    'values': [{
      'from': 'model',
      'value': '在元宇宙时代的消费将会实现从消费主体、信息获取方式、信息呈现形式以及消费体验内容等全方位实现虚实结合，同时元宇宙也将明显的改善消费者行为与心理，并为品牌与消费者互动创造全新机会。'
    }]
  })
}


const subscribeTree = ($event) => {
  const target = $event.target;
  if (target.className === 'rich-text') {
    
  } else {
    if (target.parentNode) {
      const node = searchNodeid(target);
      const fontid = target.parentNode.getAttribute('id');
      const nodeid = node.getAttribute('data-nodeid') 
      if (nodeid) {
        const __childs = [...node.childNodes];

        const __values:any = [];
        for (let i = 0; i < __childs.length; i++) {
          const v = __childs[i];
          const source = v.getAttribute('source');
          const __fontid = v.getAttribute('id');
          if (v.nodeName === 'FONT') {
            if (source === 'user') { 
              __values.push({
                'from': 'user',
                'value': v.innerText,
                'fontid': __fontid,
              })
            } else {
              if (__fontid === fontid) {
                if (tree[fontid]) {
                    const history = tree[fontid];
                    if (history !== target) {
                        console.log('需要切分')
                    }
                }
                __values.push({
                  'from': 'model',
                  'value': v.innerText,
                  'fontid': __fontid,
                })
              } else {
                __values.push({
                  'from': 'model',
                  'value': v.innerText,
                  'fontid': __fontid,
                })
              }
            }
          } else {
            __values.push({
              'from': 'user',
              'value': v.innerText,
              'fontid': __fontid,
            })
          }
        }
        //当前内容
        // const __values = __childs.map(v => {
        //   const source = v.getAttribute('source');
        //   const __fontid = v.getAttribute('id');
        //   if (v.nodeName === 'FONT') {
        //     if (source === 'user') {
        //       return {
        //         'from': 'user',
        //         'value': v.innerText,
        //         'fontid': __fontid,
        //       };
        //     } else {
        //       if (__fontid === fontid) {
        //         if (tree[fontid]) {
        //             const history = tree[fontid];
        //             if (history !== target) {
        //                 console.log('需要切分')
        //             }
        //         }
        //       }
        //       return {
        //         'from': 'model',
        //         'value': v.innerText,
        //         'fontid': __fontid,
        //       };
        //     }
        //   }
        //   return {
        //     'from': 'user',
        //     'value': v.innerText,
        //     'fontid': __fontid,
        //   };
        // })
        for (let i = 0; i < nodes.length; i++) {
          if (nodes[i].nodeid === nodeid) {
            nodes[i].values = __values;
          }
        }
      }
    }
  }
};

onMounted(() => {
  richTextRef.value.addEventListener("DOMSubtreeModified", subscribeTree, false);
  richTextRef.value.addEventListener("keydown", ($event) => {
    if ( $event.keyCode === 13 ) {
      document.execCommand('insertHTML', false, '\n');
      $event.preventDefault()
    }
  }, false);

  setTimeout(() => {
    importEditor();
  }, 10);
});

onUnmounted(() => {
  richTextRef.value.removeEventListener("DOMSubtreeModified", subscribeTree);
});


function searchNodeid(target) {
  if (target.nodeName === 'SPAN') {
    if (target.getAttribute("data-nodeid")) {
      return target;
    }
  } else if (target.nodeName === '#text' || target.nodeName === 'FONT') {
    return searchNodeid(target.parentNode);
  } else {
    return null;
  }
}



function uuid() {
    let s:any = [];
    const hexDigits = '0123456789abcdef';
    for (var i = 0; i < 36; i++) {
        s[i] = hexDigits.substr(Math.floor(Math.random() * 0x10), 1);
    }
    s[14] = '4'; // bits 12-15 of the time_hi_and_version field to 0010
    s[19] = hexDigits.substr((s[19] & 0x3) | 0x8, 1); // bits 6-7 of the clock_seq_hi_and_reserved to 01
    s[8] = s[13] = s[18] = s[23] = '-';
    return s.join('');
}
</script>
<style lang="less">
* {
  padding: 0;
  margin: 0;
}
body,
html,
.app,
#app {
  height: 100%;
  width: 100%;
}

.app {
  display: flex;
  .view {
    width: 50%;
    height: 100%;
    box-sizing: border-box;
    padding: 20px;
    overflow-y: scroll;
    &.editor {
      background: rgba(255, 0, 0, 0.31);
    }
    &.preveiw {
      background: rgba(0, 255, 40, 0.25);
    }
  }
}

.rich-text {
  width: 100%;
  height: 100%;
  border: 2px solid #fff;
  border-radius: 5px;
  padding: 20px;
  box-sizing: border-box;
  white-space: break-spaces;
  overflow-y: scroll;
}

.btns {
  > button {
    margin-left: 5px;
  }
}

font {
    &[source="user"] {
      color: #000;
    }
    &[source="model"] {
      color: red;
    }
  }

</style>
