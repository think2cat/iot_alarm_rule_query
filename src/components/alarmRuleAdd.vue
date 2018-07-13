<template>
  <div>

      <ul>
        <li v-for="(itemR,index) in item.rules">
          <div class="row">
            <div>{{index ? "并且" : "筛选"}}</div>
            <div class="form-inline">
              <select class="form-control" v-model="itemR[0].id" @change="onRuleFieldChange(itemR[0])">
                  <option v-for="obj in compFieldItems" :value="obj.id">{{obj.field}}</option>
                </select>
              <select class="form-control" v-if="item.alarmStatType==1" v-model="itemR[0].func">
                  <option v-for="(key,value) in opItem.func" :value="value">{{key}}</option>
                </select>
              <select class="form-control" v-model="itemR[0].opt">
                  <option v-for="(key,value) in getOptItemByType(itemR[0].dataType)" :value="value">{{key}}</option>
                </select>
              <!--number-->
              <input type="text" class="form-control" v-if="'between' != itemR[0].opt && 'STRING' == itemR[0].dataType" v-model="itemR[0].value" />
              <input type="number" class="form-control" v-if="'between' != itemR[0].opt && ('INT' == itemR[0].dataType || 'FLOAT' == itemR[0].dataType)" v-model="itemR[0].value" />
              <!--between-->
              <input type="number" class="form-control" v-if="itemR[0].opt == 'between'" v-model="itemR[0].minValue" />
              <input type="number" class="form-control" v-if="itemR[0].opt == 'between'" v-model="itemR[0].maxValue" />
              <!--bool & enum -->
              <select class="form-control" v-model="itemR[0].value" v-if="'BOOL' == itemR[0].dataType || 'ENUM' == itemR[0].dataType">
                  <option v-for="itemEnum in getEnumItemValueByFieldId(itemR[0].id)" :value="itemEnum">{{itemEnum}}</option>
                </select>
              <div class="pull-right" v-if="'info' != editType">
                <button class="btn btn-default" @click="addAnd(index)">并且</button>
                <button class="btn btn-default" @click="addOr(index)">或者</button>
                <button class="btn btn-danger" @click="delOr(index,0)" v-if="item.rules.length>1 || item.rules[0].length>1">删除</button>
              </div>
            </div>
          </div>
          <ul v-if="itemR.length > 1 || index < item.rules.length -1">
            <li v-for="(itemRO,indexJ) in itemR" v-if="indexJ > 0">
              <div class="row">
                <div>或者</div>
                <div class="form-inline">
                  <select class="form-control" v-model="itemRO.id" @change="onRuleFieldChange(itemRO)">
                      <option v-for="obj in compFieldItems" :value="obj.id">{{obj.field}}</option>
                    </select>
                  <select class="form-control" v-if="item.alarmStatType==1" v-model="itemRO.func">
                      <option v-for="(key,value) in opItem.func" :value="value">{{key}}</option>
                    </select>
                  <select class="form-control" v-model="itemRO.opt">
                      <option v-for="(key,value) in getOptItemByType(itemRO.dataType)" :value="value">{{key}}</option>
                    </select>
                  <!--number-->
                <input type="text" class="form-control" v-if="'between' != itemRO.opt && 'STRING' == itemRO.dataType" v-model="itemRO.value" />
                <input type="number" class="form-control" v-if="'between' != itemRO.opt && ('INT' == itemRO.dataType || 'FLOAT' == itemRO.dataType)" v-model="itemRO.value" />
                  <!--between-->
                  <input type="number" class="form-control" v-if="itemRO.opt == 'between'" v-model="itemRO.minValue" />
                  <input type="number" class="form-control" v-if="itemRO.opt == 'between'" v-model="itemRO.maxValue" />
                  <!--bool & enum -->
                  <select class="form-control" v-model="itemRO.value" v-if="'BOOL' == itemRO.dataType || 'ENUM' == itemRO.dataType">
                  <option v-for="itemEnum in getEnumItemValueByFieldId(itemRO.id)" :value="itemEnum">{{itemEnum}}</option>
                </select>
                  <div class="pull-right" v-if="'info' != editType">
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
//  import api from "@/api";
//import select2 from "../../../node_modules/select2/dist/js/select2";
import $ from "jquery";
export default {
  data() {
    return {
      activeTab: 0,
      editType: "",
      menuItems: ["设置告警规则", "通知方式", "应用范围"],
      alarmLevelItems: [], //TODO
      opItem: {
        statPeriod: [1, 3, 5, 10, 20, 30, 60],
        repeatPeriod: [5, 10, 15, 30, 60, 120],
        repeatPeriodUnit: ["分钟内", "小时内", "累计"],
        repeatTime: [1, 2, 3, 4, 5, 10],
        alarmInterval: {
          "0": "不静默",
          "1": "1分钟",
          "5": "5分钟",
          "10": "10分钟",
          "15": "15分钟",
          "30": "30分钟",
          "60": "60分钟",
          "999": "永久静默"
        },
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
      contactGroupItems: [],
      btnDom: [],
      tabMenu: [],
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
        alarmDevices: [
          {
            deviceGroupId: "",
            scope: 0
          }
        ],
        alarmReceivers: [
          {
            alarmInterval: 0,
            email: 0,
            ems: 0,
            userGroupId: ""
          }
        ],
        tProductId: "",
        name: "",
        description: "",
        alarmLevel: 0,
        alarmStatType: 0,
        statPeriod: 1,
        repeatPeriod: 10,
        repeatPeriodUnit: 0,
        repeatTime: 3,
        status: 0,
        rules: []
      }
    };
  },
  mounted() {
    /*
      this.tabMenu = $(this.$el).find("ul.nav > li");
      this.btnDom = $(this.$el).find(".box-footer button:gt(0)");
      this.switchTab(this.activeTab);
      this.editType = "add";
      */
    //this.contentDom = $(this.$el).find(".box-body > div").children();
    //TODO
    /*
      let dom = $(this.$el).find("select[multiple]");
      let dom2 = dom.select2();
      dom.on("select2:unselect", v => {
        //更新值
      });
      dom.on("select2:select", v => {
        //更新值
      });
      */
  },
  computed: {
    compFieldItems() {
      if (1 == this.item.alarmStatType) {
        return this.fieldItems.filter(v => {
          return "INT" == v.dataType || "FLOAT" == v.dataType;
        });
      } else {
        return this.fieldItems;
      }
    }
  },
  methods: {
    switchTab(val) {
      //标签页切换
      if (
        "info" != this.editType &&
        val > this.activeTab &&
        !this.checkData()
      ) {
        return;
      }
      this.activeTab = val;
      this.tabMenu
        .removeClass("active")
        .eq(this.activeTab)
        .addClass("active");
      this.checkEditType();
      /*
        $(this.$el)
          .find("select[multiple='']")
          .select2();
          */
    },
    getFieldList(callbcak) {
      /*
        api.getProductfield(this.item.tProductId, response => {
          if (0 == response["code"] && response["data"]) {
            this.fieldItems = response["data"];
            if (this.fieldItems.length && "info" != this.editType) {
              //如果是新增和编辑
              this.emptyItemRules = this.field2Rule(this.fieldItems[0]);
              if ("add" == this.editType) {
                //如果是新增
                this.item.rules.push([$.extend(true, {}, this.emptyItemRules)]);
              }
            }
            //console.log("this.fieldItems", this.fieldItems);
          }
          callbcak && callbcak(response);
        });
        */
    },
    checkEditType() {
      if ("info" == this.editType) {
        //查看规则则禁用所有表单
        setTimeout(() => {
          this.contentDom = $(this.$el).find(".box-body > div > div");
          this.contentDom.find(":input").attr("disabled", "");
        }, 10);
      }
    },
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
      //数据点下拉框改版时触发
      //console.log("onRuleFieldChange", item.name);
      for (let i = 0; i < this.compFieldItems.length; i++) {
        if (item.id == this.compFieldItems[i].id) {
          item = $.extend(true, item, this.field2Rule(this.compFieldItems[i]));
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
    },
    //提交数据
    submit() {
      /*
        if (1 != this.publishState) {
          return;
        }
        if (!this.checkData()) {
          this.activeTab = 0;
          return;
        }
        */
      var cb = response => {
        if (0 == response["code"]) {
          //提交成功
          // TODO "成功";
        } else {
          //提交失败
        }
      };
      //console.log("sumbit", this.item.params);
    }
  }
};
</script>

<style scoped lang="less">
@import "../../node_modules/bootstrap/dist/css/bootstrap.min.css";
.box-body {
  @icon_height: 50px;
  @icon_bg_color: #669df0;
  > ul.nav {
    //顶部1-2-3
    border-bottom: none;
    margin: 0;
    width: 100%;
    display: flex;
    > li {
      text-align: center;
      flex-grow: 1;
      height: 57px;
      padding: 0;
      margin: 0;
      border-bottom: 3px solid #f7f7f7;
      text-align: center;
      > div {
        width: 100%;
        position: absolute;
        color: #ccc;
        font-size: 22px;
      }
      &.active {
        border-bottom: 3px solid @icon_bg_color;
        > div:nth-child(1) {
          > span {
            background-color: @icon_bg_color;
            color: #fff;
          }
        }
        > div:nth-child(2) {
          color: @icon_bg_color;
        }
      }
      > div:nth-child(1) {
        top: 50%;
        > span {
          line-height: @icon_height;
          height: @icon_height;
          width: @icon_height;
          border-radius: 50%;
          background-color: #f7f7f7;
          font-size: 30px;
          display: inline-block;
        }
      }
      > div:nth-child(2) {
        line-height: 15px;
      }
    }
  }
  > div {
    //表单区
    margin: 50px 40px 20px 40px;
    min-height: 398px;
    > div {
      //display: none;
      .row {
        padding: 3px;
        margin-left: 0;
        margin-right: 0;
      }
    }
    > .rule {
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
    > .notice {
      //通知方式
      ul {
        list-style: none;
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

  //select[multiple] + span.select2 {
  //  border: 1px solid red;
  //}
}
</style>
