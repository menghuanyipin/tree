<template>
  <div>
    <div id="treeMount"></div>
  </div>
</template>
<script>
import $ from "jquery";
export default {
  name: "Tree",
  /**
   * data ： 数据源
   * extend ： 默认是否展开树，false不展开
   * hideTopNode ：是否隐藏顶级节点，true表示隐藏
   * showCheckbox ： 是否显示复选框
   * height ：ul的高度，默认为300px
   */
  props: {
    data: {
      type: Array,
      default: function () {
        return [];
      },
      required: true,
    },
    height: {
      type: Number,
      default: 300,
    },
    extend: {
      type: Boolean,
      default: false,
    },
    hideTopNode: {
      type: Boolean,
      default: false,
    },
    showCheckbox: {
      type: Boolean,
      default: false,
    },
  },
  data() {
    return {
      // 展开和收起tree的设置，来自props的extend
      treeExtend: false,
      // 所有checkbox选中的项目
      treeData: [],
    };
  },
  methods: {
    // 初始化Tree目录
    init() {
      //  如果顶级节点隐藏，那么必须展开树，也就是说extend属性的设置失效，默认为true
      if (this.hideTopNode == true) {
        this.treeExtend = true;
      }
      // 根节点
      let ul = $("<ul id='tree'></ul>");
      // 递归生成目录树
      this.generatorTree(this.data, ul);
      // 如果需要隐藏根节点，则取出根节点下面的第一个ul作为根节点
      if (this.hideTopNode) {
        ul = $(ul.find("ul")[0]);
      }
      // 取消顶级ul的padding-left
      ul.css({
        "padding-left": "0px",
        "list-style": "none",
        "overflow-y": "auto",
        height: `${this.height}px`,
      });
      // 挂载tree目录
      $("#treeMount").empty();
      $("#treeMount").append(ul);
    },
    /**
     * 监听checkbox的变化，主要是子box都选中后，父box默认进入全选状态；
     * 否则任意一个子box的状态为未选中，父box为反选
     */
    watchCheckbox(checkbox) {
      // 处理选中样式的变化
      let li = $(checkbox).parent("li").parentsUntil("li").parent();
      let checked = true;
      for (let input of li.find("li").find("input")) {
        if ($(input).prop("checked") == false) {
          checked = false;
          break;
        }
      }
      $(li.children("input")).prop("checked", checked);
    },
    /**
     * 获取tree目录上所有选中的节点的数据
     */
    watchTreeData() {
      let inputs = $("input:checked");
      this.treeData = [];
      for (let i of inputs) {
        let id = $(i).val();
        this.findTreeData(id, this.data);
      }
    },
    /**
     * 递归遍历tree目录，搜索符合当前节点的数据
     */
    findTreeData(id, data) {
      for (let item of data) {
        if (item.id == id) {
          this.treeData.push(item);
        } else {
          this.findTreeData(id, item.children);
        }
      }
    },
    // 递归生成树
    generatorTree(data, container) {
      for (let item of data) {
        let li = $("<li></li>");
        // 增加id属性
        li.attr("id", item.id);
        // 如果有checkbox，处理checkbox的全选，反选等事件
        if (this.showCheckbox) {
          let box = $(`<input type='checkbox' value=${item.id}>`);
          box.css({
            width: "16px",
            height: "16px",
            "vertical-align": "top",
          });
          box.on("click", (event) => {
            let checkbox = event.target;
            let li = $(checkbox).parent("li");
            // 全选和反选
            if ($(checkbox).prop("checked")) {
              li.find("input").prop("checked", true);
            } else {
              li.find("input").prop("checked", false);
            }
            // 监听每一次checkbox的变化
            this.watchCheckbox(checkbox);
            this.watchTreeData();
          });
          li.append(box);
        }
        // 向外传递的事件
        // 如果有复选框，则给复选框添加单击事件，否则给li添加
        if (this.showCheckbox) {
          li.on("click", "input", (event) => {
            event.stopPropagation();
            if ($(event.target).prop("checked")) {
              this.$emit("row", item);
            } else {
              this.$emit("row", {});
            }
            this.$emit("rows", this.treeData);
          });
        } else {
          li.on("click", (event) => {
            event.stopPropagation();
            this.$emit("row", item);
          });
        }

        // extend为false表示收起所有折叠，true表示展开
        if (this.treeExtend == false) {
          li.css({ height: "22.5px", overflow: "hidden" });
        }
        // 增加标签内容
        let span = $("<span></span>");
        // 计算标签的宽度
        let spanWidth = 0;
        if (item.children && item.children.length > 0) {
          spanWidth += 23;
        }
        if (this.showCheckbox) {
          spanWidth += 23;
        }

        span.css({
          display: "inline-block",
          width: `calc(100% - ${spanWidth}px)`,
          "vertical-align": "middle",
          overflow: "hidden",
          "text-overflow": "ellipsis",
          "white-space": "nowrap",
          "text-align": "left",
        });
        span.text(item.label);
        li.append(span);
        container.append(li);

        // 判断是否有孩子
        if (item.children && item.children.length > 0) {
          // 生成上下的箭头
          let img = $("<img />");
          img.attr(
            "src",
            "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAMgAAADICAYAAACtWK6eAAAMcUlEQVR4Xu2dSawuRRmGX5xwnucRxQlH0CDRGG80BjVGAzFg4sqlW1zJzh3uWLpw4dIEjNFojGgMRoMKccBZFBXnAecZ57zeOvEg95zTXd3VX31dT63/6qp6vnr+qu6uqj5LJAhA4EgCZ8EGAhA4mgCC0DsgcAwBBKF7QABB6AMQqCPACFLHjVyDEECQQQJNM+sIIEgdN3INQgBBBgk0zawjgCB13Mg1CAEEGSTQNLOOAILUcSPXIAQQZJBA08w6AghSx41cgxBAkEECTTPrCCBIHTdyDUIAQQYJNM2sI4AgddzINQgBBBkk0DSzjgCC1HEj1yAEEGSQQNPMOgIIUseNXIMQQJBBAk0z6wggSB03cg1CAEEGCTTNrCOAIHXcyDUIAQQZJNA0s44AgtRxI9cgBBBkkEDTzDoCCFLHjVyDEECQQQJNM+sIIEgdN3INQgBBBgk0zawjgCB13Mg1CAEEGSTQNLOOAILUcSPXIAQQZJBA08w6AghSx41cgxBAkEECTTPrCCBIHTdyDUIAQQYJNM2sI4AgddzINQgBBBkk0DSzjgCC1HEj1yAEEGSQQNPMOgIIUseNXIMQQJBBAk0z6wggSB03cg1CAEEGCTTNrCOAIHXcyDUIgbUEeYGkU5IulHSepHMk3U3SDyXdIul6SZ+W9IVBuNLMtgTc3y6W9GJJ50p6gqR/SbpN0uck3SDpOkk/XVqNNQS5StIVks4+oTJ3SLpa0pVLK03+oQlM7W8/kfQuSW9fQmupINdIumxmBW6SdNHMPPwcAiaweX9bIsi/F8ZsSdkLiyZ7QgIh/a22k94o6UULIbvBd5e0tOELq0H2zgm4j/5TUm1fPWjetZIun9vWmkI9B3zb3IKO+L1vrO5ZbrBWuiSX2REBP+j5xwpyHCB5x9x74LmC+OmBn0addEM+J0b+d7gXksxBNsRvPbv4W3kaulaD/aDoJXOeps4V5C2S3rlWbQ9dx/8S9y5DaYPLc8lkBO4hyZ3ZI8ja6a3laeqk684V5N2S3jzpyvN/9HdJ90GS+eB2lsNT7r+U+9MWTXuPpDdNvfBcQb4o6fypF6/4nSW5b5l3VmQnS3ICnmr/SZJHkFbpZkkXTL34XEF+I+nBUy9e+TvPO+8vybKQxiHg+9o/NpbDNH8r6SFTsfYoiOvu+ecDkGRqGNP/zvefvy9PNFs3pqkgradYh+H8VdKDypOM1tC4fhwB33f+biM53MqmU6yWN+lnCpFv1jyl87SLtD8Cvt/0tN33HlulpjfprR7zHgfnz5IeWqZdW0GknPYELMevV36nNqXWTR/ztnhROKVRfrLxcEmedpHyE7ifpF8FyNH8RaFDs+ZSkzmh9hOORyDJHGRd/tZPKG8vL4a3rmDzpSYHDVpjsWINnD9IelR5kVSTnzyxBPxk8hdBcmy2WPEAcdQqXD8OfDSSxPb0itIfKOlnZbVERfbFWea+0vhvgVWZDlU1UpLHSPINPKl/An5c7x1+vjGPSNX9vDpjB5L42fnjytKECOiUOY2AH9P/OKMca4wg0dMtvxV9PJJM66kBv/KSDh/c4adWEWnxALD4Ah2MJH7R9MSyjiciCJR5ZgJ+d/WDzHKsOYJEjyR+4fQkJOnG1YeVI3j8SDcirfbHv9qFOhhJ/OLpyZL8KJgUR8AvdL9bFptG1GLVPr3qxTqQ5JflIDE/CiZtT8Avcr+zFzlaTLEOhyTqEbDf0j61LJ/evouMW6LluFWS33dEpCZ/9k0u2sFI4re1Ty/LqCOCNVqZj5T07b3J0XoEib5x/7mkZyBJc1e99MfnL/tlYERq+iff9OIdjCSW5Jllm2VE8PZeppf8fGODbdhHcWzef5sX0IEkXv/zrLIxZ+8ddsv2eanP1/csx1ZTrB5u3H0M/rORZDV/Hivpq3MOP1it5NMX2uyPfbOCOhhJvFjuuWUX28rxGupyXv/2lRHk2NTE/+tCUY+AvWju+WU321C9eqXGet3bl8oW6JUuOesym/+hb15gByPJj8rhd37zTppOwF9x8qk2XkYSkUL6akihHUjiFabeX+8376STCXgxqD+fN5QckVOsHm7cvdL0hUhyoh1eBOrv/nmNVUQK/RMPLbyDkeT75cOjXp5CuiuBoeXoZQQ5CEvUjbu/jOpvJnp5Cul/BPylYn9P0musIlIXf95dVKKDkQRJ7qyAtw18VpLXWEWkbvplNxXpQJLvle9ue3nKyOkpkj6DHKe7QI+CuF5R0y1v9PEnukaV5FxJN5SzxyL+JLrrj91VqIORxBt+XlrOcIroJFFleg/Np8qZYxF16LIvdlmpDiTxxp+XSfIarhHS0yR9EjnuGureBYmcbnkD0KkBJPHGsk9I8urciNR1H+y6ch2MJN+S9PJyKmBE52ldpjeUXY8cR2POIkjkSOLdcq/YoSTeSPZxSV66HpFS9L0UlexgJPmmpFeWIzQjOtPaZZ4n6WPl6Na1rz3lemn6XZqKdiCJt5ZeLMmrgTMn7678KHJMC2FGQSKnW95i+qrEkliO68p5xtN6yLq/Stff0lW4g5HEkry6HMq8bvdpezVvOf4IcsyDnFmQyJHka5Jek0iS50j6sCRveopIaftZ2op3MJL40ILXlhPMIzrd1DK9D/9D5QT8qXnW/F3qPpa68h1I4sMLXifJ+0p6TM+T9EHkqA/NXgSJnG59WdLrO5TEh1N8oHwWor6H1OfcRd/aRSM6GEl80scl5ZsY9V1qvZznS3o/ciwHujdBIkeSmyVd2oEkF0h6nyTvCIxIu+pTu2pMByOJj8V5gyRvvopIPqnlveVDQhHl764/7a5BHUji43EuK19Z2rKT+oSWa5FjXeR7FiRyuvV5SZdvKInluEaSt8tGpN32o902rIORxJK8sXySrGWn9bTKIwdyNKA8giCRI4mnW5bEOxRbJN+QWw7vJY9Iu+8/u29gByOJb9w93VpbEsvhaZX3kkekIfrOEI3sRBKPJN7Gu0byew6PHMixBs1jrjGaIJHTLb8n8UiyVBLL4ZHDBy1EpKH6zFCN7WAksSQeSbzXvSZ5+YhHDuSooVeRZ1RBIkcSL0uxJN7rPid54aHl8CkkEWnIvjJkozsYSbzA0dOtqZJYDk+rfApJRBq2nwzb8E4k8UjiAyGOS97P4ZEDOQL+GhDkNPSos4C9n8QjyVGSWA6PHD6iJyIN3z+GB9DBSGJJPJL41JTDydtkPXIgR8RfQykTQe4MP2ok8fZdS+IDIZx8wILl8PlVEYl+gSBH9rsoSXwQhKdb7pyeVvmInoiEHIeoA+PMXTBSEscEOSL+Gs5QJoIcHYgoSaK6Bn0BQWb3vVEkQY4jugZgTnZm75LQB47pA8A5WRD/Yq+SEP8T4g+gaYLsURJiPyH2QJoA6dBP9jKSEPeJcQfURFA7koSYz4g5sGbA2oEkxHtmvAE2E1hiSYh1RayBVgEtoSTEuTLOgKsEl0gSYrwgxsBbAC+BJMR3YXwBuBBgx5IQ2xViC8QVIHYoCXFdKa6AXAlkR5IQ0xVjCswVYXYgCfFcOZ4AXRlooCTEskEsgdoAaoAkxLFRHAHbCOyGkhDDhjEEbkO4G0hC/BrHD8CNATeUhNhtEDsgbwC5gSTEbaO4AXoj0CtKQsw2jBmwN4R9qCgfDOdPRc9JN0m6aE4GfrucAIIsZ1h7haskXSHp7BMucIekqyVdWVsQ+eoJIEg9uzVy+hPOpyRdWM7hPadc9DZJ/hrVjZI8cvhruaQAAggSAJ0i8xBAkDyxoqYBBBAkADpF5iGAIHliRU0DCCBIAHSKzEMAQfLEipoGEECQAOgUmYcAguSJFTUNIIAgAdApMg8BBMkTK2oaQABBAqBTZB4CCJInVtQ0gACCBECnyDwEECRPrKhpAAEECYBOkXkIIEieWFHTAAIIEgCdIvMQQJA8saKmAQQQJAA6ReYhgCB5YkVNAwggSAB0isxDAEHyxIqaBhBAkADoFJmHAILkiRU1DSCAIAHQKTIPAQTJEytqGkAAQQKgU2QeAgiSJ1bUNIAAggRAp8g8BBAkT6yoaQABBAmATpF5CCBInlhR0wACCBIAnSLzEECQPLGipgEEECQAOkXmIYAgeWJFTQMIIEgAdIrMQwBB8sSKmgYQQJAA6BSZhwCC5IkVNQ0ggCAB0CkyDwEEyRMrahpAAEECoFNkHgIIkidW1DSAAIIEQKfIPAQQJE+sqGkAAQQJgE6ReQggSJ5YUdMAAggSAJ0i8xBAkDyxoqYBBP4D++td2IM+EFkAAAAASUVORK5CYII="
          );
          img.css({
            width: "15px",
            height: "15px",
            margin: "3px 3px 3px 4px",
            "vertical-align": "middle",
            // 箭头展开折叠的角度变化
            transform: this.extend ? "rotate(0deg)" : "rotate(-90deg)",
          });
          img.on("click", (event) => {
            event.stopPropagation();
            //   折叠处理
            if (li.css("height") == "22.5px") {
              li.css({ height: "auto" });
              img.css({
                transform: "rotate(0deg)",
              });
            } else {
              li.css({ height: "22.5px", overflow: "hidden" });
              img.css({
                transform: "rotate(-90deg)",
              });
            }
          });
          li.prepend(img);
          // 生成孩子列表
          let ul = $("<ul></ul>");
          ul.css({ "list-style": "none", overflow: "hidden" });
          li.append(ul);
          this.generatorTree(item.children, ul);
        }
      }
    },
  },
  watch: {
    data(val) {
      if (val && val.length > 0) {
        this.init();
      }
    },
  },
  created() {
    this.treeExtend = this.extend;
  },
  mounted() {
    this.init();
  },
};
</script>
<style lang="less" scoped>
ul,
li {
  list-style: none;
}
#treeMount {
  font-size: 16px !important;
}
/deep/::-webkit-scrollbar {
  /*滚动条整体样式*/
  width: 3px; /*高宽分别对应横竖滚动条的尺寸*/
  height: 1px;
}
/deep/::-webkit-scrollbar-thumb {
  /*滚动条里面小方块*/
  border-radius: 3px;
  box-shadow: inset 0 0 5px rgba(0, 0, 0, 0.2);
  background: #aaa;
}
/deep/::-webkit-scrollbar-track {
  /*滚动条里面轨道*/
  box-shadow: inset 0 0 5px rgba(0, 0, 0, 0.2);
  border-radius: 10px;
  background: #ededed;
}
</style>