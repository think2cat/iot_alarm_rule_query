<template>
  <div class="rule">
      <ul>
        <li v-for="(itemR,index) in item.rules" :key="itemR[0].id">
          <div class="row">
            <div>{{index ? "并且" : "筛选"}}</div>
            <div class="form-inline">
              <select class="form-control" v-model="itemR[0].id" @change="onRuleFieldChange(itemR[0])">
                  <option v-for="obj in compFieldItems" :value="obj.id" :key="obj.id">{{obj.field}}</option>
                </select>
              <select class="form-control" v-model="itemR[0].opt">
                  <option v-for="(key,value) in getOptItemByType(itemR[0].dataType)" :value="value" :key="value">{{key}}</option>
                </select>
              <!--number-->
              <input type="text" class="form-control" v-if="'between' != itemR[0].opt && 'STRING' == itemR[0].dataType" v-model="itemR[0].value" />
              <input type="number" class="form-control" v-if="'between' != itemR[0].opt && ('INT' == itemR[0].dataType || 'FLOAT' == itemR[0].dataType)" v-model="itemR[0].value" />
              <!--between-->
              <input type="number" class="form-control" v-if="itemR[0].opt == 'between'" v-model="itemR[0].minValue" />
              <input type="number" class="form-control" v-if="itemR[0].opt == 'between'" v-model="itemR[0].maxValue" />
              <!--bool & enum -->
              <select class="form-control" v-model="itemR[0].value" v-if="'BOOL' == itemR[0].dataType || 'ENUM' == itemR[0].dataType">
                  <option v-for="itemEnum in getEnumItemValueByFieldId(itemR[0].id)" :value="itemEnum" :key="itemEnum">{{itemEnum}}</option>
                </select>
              <div class="pull-right">
                <button class="btn btn-default" @click="addAnd(index)">并且</button>
                <button class="btn btn-default" @click="addOr(index)">或者</button>
                <button class="btn btn-danger" @click="delOr(index,0)" v-if="item.rules.length>1 || item.rules[0].length>1">删除</button>
              </div>
            </div>
          </div>
          <ul v-if="itemR.length > 1 || index < item.rules.length -1">
            <li v-for="(itemRO,indexJ) in itemR" v-if="indexJ > 0" :key="itemRO.id">
              <div class="row">
                <div>或者</div>
                <div class="form-inline">
                  <select class="form-control" v-model="itemRO.id" @change="onRuleFieldChange(itemRO)">
                      <option v-for="obj in compFieldItems" :value="obj.id" :key="obj.id">{{obj.field}}</option>
                    </select>
                  <select class="form-control" v-model="itemRO.opt">
                      <option v-for="(key,value) in getOptItemByType(itemRO.dataType)" :value="value" :key="value">{{key}}</option>
                    </select>
                  <!--number-->
                <input type="text" class="form-control" v-if="'between' != itemRO.opt && 'STRING' == itemRO.dataType" v-model="itemRO.value" />
                <input type="number" class="form-control" v-if="'between' != itemRO.opt && ('INT' == itemRO.dataType || 'FLOAT' == itemRO.dataType)" v-model="itemRO.value" />
                  <!--between-->
                  <input type="number" class="form-control" v-if="itemRO.opt == 'between'" v-model="itemRO.minValue" />
                  <input type="number" class="form-control" v-if="itemRO.opt == 'between'" v-model="itemRO.maxValue" />
                  <!--bool & enum -->
                  <select class="form-control" v-model="itemRO.value" v-if="'BOOL' == itemRO.dataType || 'ENUM' == itemRO.dataType">
                  <option v-for="itemEnum in getEnumItemValueByFieldId(itemRO.id)" :value="itemEnum" :key="itemEnum">{{itemEnum}}</option>
                </select>
                  <div class="pull-right">
                    <button class="btn btn-danger" @click="delOr(index,indexJ)">删除</button>
                  </div>
                </div>
              </div>
            </li>
          </ul>
        </li>
      </ul>
    </div>
</template>

<script>
//  import api from '@/api'
import $ from "jquery";
import requireField from "./field.json";
export default {
  data() {
    return {
      opItem: {
        func: {
          avg: "平均值",
          min: "最小值",
          max: "最大值"
        },
        //当数据点类型为枚举或布尔时，运算符只能为 等于 或 不等于 ， 且对比值为下拉框而非输入框。
        //当数据点类型为字符串型，运算符智能为包含和不包含
        //其他类型，运算符为>、<、 >=、 <=、  = 、!=、在...之间
        op: {
          BOOL: {
            "==": "==",
            "!=": "!="
          },
          STRING: {
            //in: "包含",
            //noin: "不包含",
            "==": "==",
            "!=": "!="
          },
          OTHER: {
            ">": ">",
            ">=": ">=",
            "<": "<",
            "<=": "<=",
            "==": "==",
            "!=": "!=",
            between: "在...之间"
          }
        }
      },
      fieldItems: [],
      btnDom: [],
      contentDom: [],
      emptyItemRules: {
        id: "",
        field: "",
        dataType: "",
        func: "avg",
        opt: ">",
        value: "",
        maxValue: "",
        minValue: ""
      },
      item: {
        rules: []
      }
    };
  },
  mounted() {
    setTimeout(() => {
      this.fieldItems = requireField.data.list;
      this.item.rules.push([$.extend(true, {}, this.emptyItemRules)]);
    }, 100);
  },
  computed: {
    compFieldItems() {
      // if (1 == this.item.alarmStatType) {
      //   return this.fieldItems.filter(v => {
      //     return "INT" == v.dataType || "FLOAT" == v.dataType;
      //   });
      // } else {
        return this.fieldItems;
      // }
    }
  },
  methods: {
    getOptItemByType(val) {
      //传入dataType返回对应操作符
      if ("BOOL" == val || "ENUM" == val) {
        return this.opItem.op.BOOL;
      } else if ("STRING" == val) {
        return this.opItem.op.STRING;
      } else {
        return this.opItem.op.OTHER;
      }
    },
    getEnumItemValueByFieldId(id) {
      //console.log("getEnumItemValueByFieldId", id);
      //传入数据点ID返回对应枚举值或布尔值
      for (let i = 0; i < this.fieldItems.length; i++) {
        if (id == this.fieldItems[i].id) {
          if ("BOOL" == this.fieldItems[i]["dataType"]) {
            return [true, false];
          } else if ("ENUM" == this.fieldItems[i]["dataType"]) {
            return this.fieldItems[i]["expression"].split(";");
          }
        }
      }
    },
    field2Rule(fieldItem) {
      //console.log("field2Rule", fieldItem.field);
      let ret = $.extend(true, {}, this.emptyItemRules);
      ret.id = fieldItem.id;
      ret.field = fieldItem.field;
      ret.dataType = fieldItem.dataType;
      let optArr = this.getOptItemByType(fieldItem.dataType);
      for (let i in optArr) {
        ret.opt = i;
        break;
      }
      if ("BOOL" == fieldItem.dataType || "ENUM" == fieldItem.dataType) {
        let valueArr = this.getEnumItemValueByFieldId(fieldItem.id);
        if (valueArr.length) {
          ret.value = valueArr[0];
        }
      } else {
        ret.value = "";
      }
      return ret;
    },
    onRuleFieldChange(item) {
      //数据点下拉框改变时触发
      // console.log("onRuleFieldChange", item);
      for (let i = 0; i < this.compFieldItems.length; i++) {
        if (item.id == this.compFieldItems[i].id) {
          item = $.extend(true, item, this.field2Rule(this.compFieldItems[i]));
          // console.log('onRuleFieldChange.find', item)
          return;
        }
      }
    },
    addAnd(val) {
      this.item.rules.push([$.extend(true, {}, this.emptyItemRules)]);
    },
    addOr(val) {
      this.item.rules[val].push($.extend(true, {}, this.emptyItemRules));
    },
    delOr(i, j) {
      this.item.rules[i].splice(j, 1);
      if ($.isEmptyObject(this.item.rules[i])) {
        this.item.rules.splice(i, 1);
      }
    },
    checkData() {
      this.contentDom = $(this.$el).find(".box-body > div > div");
      var inputArr = this.contentDom.find(":input");
      if (0 == this.activeTab) {
        if ("" == this.item.name) {
          //TODO "请输入名称";
          return false;
        } else if (this.item.rules.length) {
          for (let i = 0; i < this.item.rules.length; i++) {
            for (let j = 0; j < this.item.rules[i].length; j++) {
              let tmpItem = this.item.rules[i][j];
              if ("BOOL" != tmpItem.dataType) {
                let tmpDom = this.contentDom.find(
                  "ul > li:eq(" + i + ") div.row:eq(" + j + ") :text:last"
                );
                if (
                  ("INT" == tmpItem.dataType || "FLOAT" == tmpItem.dataType) &&
                  (("between" != tmpItem.opt && !$.isNumeric(tmpItem.value)) ||
                    ("between" == tmpItem.opt &&
                      (!$.isNumeric(tmpItem.minValue) ||
                        !$.isNumeric(tmpItem.maxValue))))
                ) {
                  // TODO "请检查输入值";
                  return false;
                }
                if (
                  ("between" != tmpItem.opt && !tmpItem.value) ||
                  ("between" == tmpItem.opt &&
                    (!tmpItem.minValue ||
                      !tmpItem.maxValue ||
                      parseFloat(tmpItem.minValue) >
                        parseFloat(tmpItem.maxValue)))
                ) {
                  // TODO "请检查输入值";
                  return false;
                }
              }
            }
          }
        }
      }
      return true;
      for (let x in this.item) {
        if ("string" == typeof this.item[x]) {
          this.item[x] = this.item[x].trim();
        }
      }
      var inputArr = $(this.$el).find(":input");
      /*
        if ("" == this.item.command) {
          // TODO "请输入指令名";
          return false;
        } else if (!api.tool.test(this.item.command, "fieldname")) {
          // TODO "指令名支持英文字母、数字和下划线，并以英文开头";
          return false;
        } else if ("" == this.item.name) {
          // TODO "请输入显示名称";
          return false;
        } else if (!api.tool.test(this.item.name, "nickname")) {
          // TODO "请检查显示名称";
          return false;
        }
        */
      return true;
    }
  }
};
</script>

<style scoped lang="less">
@import "../../node_modules/bootstrap/dist/css/bootstrap.min.css";
@icon_height: 50px;
@icon_bg_color: #669df0;
.rule {
  width: 800px;
  //规则
  ul {
    list-style: none;
    padding: 0;
    > li {
      .row {
        background-color: #f7f7f7;
        display: flex;
        padding: 0;
        margin: 0;
        > div {
          display: inline-block;
          padding: 3px;
        }
        > div:nth-child(1) {
          flex-grow: 0;
          min-width: 40px;
          background-color: @icon_bg_color;
          text-align: center;
          color: #fff;
        }
        > div:nth-child(2) {
          flex-grow: 1;
          padding-left: 2em;
        }
      }
      > ul {
        //子规则
        margin-left: 20px;
        padding: 0 0 0.5em 20px;
        list-style: none;
        padding-bottom: 10px;
        border-left: 5px solid @icon_bg_color;
        > li {
          padding: 1px 0 1px 0;
          > .row > div:nth-child(1) {
            background-color: #669df082;
          }
        }
      }
    }
    input[type="text"],
    input[type="number"] {
      width: 5em;
    }
  }
}
select.form-control,
input.form-control {
  height: 30px;
  padding: 3px 6px;
}
input.form-control[type="checkbox"]:focus {
  box-shadow: none;
}
.form-inline{
  text-align: left;
}
.pull-right{
  float:right;
  button {
    margin: auto 1em;
  }
}

//select[multiple] + span.select2 {
//  border: 1px solid red;
//}
</style>
