<template>
  <el-form ref="form" class="form" label-position="top">
    <div>
      <el-text class="mx-1">{{ $t("UI.title") }}</el-text>
      <el-link
        href="https://element.eleme.io"
        type="primary"
        style="margin-left: 2vw"
        >{{ $t("UI.tutorial") }}</el-link
      >
    </div>

    <!-- 
    <el-alert
      style="margin: 20px 0; background-color: #e1eaff; color: #606266"
      :title="$t('alerts.selectNumberField')"
      type="info"
    /> -->

    <el-divider content-position="center">{{ $t("UI.jd_account") }}</el-divider>
    <div style="text-align: center">
      <el-button type="primary" style="margin-top: 10px; width: 40%"
        >{{ $t("UI.jd_login") }}</el-button
      >
    </div>
    <el-divider content-position="center">{{ $t("UI.query_settings") }}</el-divider>

    <div class="block">
      <el-text class="time">{{ $t("UI.start_time") }}</el-text>
      <el-date-picker
        v-model="timeStart"
        type="datetime"
        :placeholder="$t('placeholder.start_time')"
      >
      </el-date-picker>
    </div>
    <div style="margin-top: 10px">
      <el-text class="time">{{ $t("UI.end_time") }}</el-text>
      <el-date-picker
        v-model="timeEnd"
        type="datetime"
        :placeholder="$t('placeholder.end_time')"
      >
      </el-date-picker>
    </div>
    <el-divider content-position="center">{{ $t("UI.select_settings") }}</el-divider>
    <div style="margin-top: 10px">
      <!-- <el-form-item :label="$t('labels.number')" size="large" required>
        <el-select
          v-model="numberFieldId"
          :placeholder="$t('placeholder.number')"
          style="width: 100%"
        >
          <el-option
            v-for="meta in fieldListSeView"
            :key="meta.id"
            :label="meta.name"
            :value="meta.id"
          />
        </el-select>
      </el-form-item> -->

      <!-- 多选框 -->
      <el-checkbox
        v-model="checkAll"
        :indeterminate="isIndeterminate"
        @change="handleCheckAllChange"
      >
        {{ $t("UI.checkAll") }}
      </el-checkbox>
      <el-checkbox-group
        v-model="checkedbill"
        @change="handleCheckedbillChange"
      >
        <el-checkbox
          v-for="billinfo in bill"
          :key="billinfo"
          :label="billinfo"
          :value="billinfo"
        >
          {{ $t(`info.${billinfo}`) }}
        </el-checkbox>
      </el-checkbox-group>
    </div>

    <el-button
      v-loading="isWritingData"
      @click="getData"
      color="#3370ff"
      type="primary"
      plain
      size="large"
      style="margin-top: 20px"
      >{{ $t(`UI.writeData`) }}</el-button
    >
    <!-- <el-button type="primary" plain size="large" style="margin-top: 20px"
      >新增一行记录</el-button
    > -->
  </el-form>
</template>



<script lang="ts" setup>
import { bitable, FieldType } from "@lark-base-open/js-sdk";
import { useI18n } from "vue-i18n";
import { ref, onMounted } from "vue";
import axios from "axios";

// -- 数据区域
const { t } = useI18n();
const interfaceAddr = ref("https://wuliu.market.alicloudapi.com/kdi");
const appcode = ref("db7bd581534a4da2b13f60713ff83396");
const timeRange = ref("");
// 多选框
const checkAll = ref(false);
const isIndeterminate = ref(true);
const checkedbill = ref(["Name", "BillID", "Price"]);
const bill = [
  "Name",
  "BillID",
  "Price",
  "Time",
  "invoice_link",
  "product_link",
];
const timeStart = ref();
const timeEnd = ref();

const handleCheckAllChange = (val: boolean) => {
  checkedbill.value = val ? bill : [];
  isIndeterminate.value = false;
};
const handleCheckedbillChange = (value: string[]) => {
  const checkedCount = value.length;
  checkAll.value = checkedCount === bill.length;
  isIndeterminate.value = checkedCount > 0 && checkedCount < bill.length;
};

// -- 核心算法区域
const getLocsInfo = async () => {
  const url = `${interfaceAddr.value}?no=${IsbnValue.value}`;

  const config = {
    headers: {
      Authorization: `APPCODE ${appcode.value}`,
      "Content-Type": "application/json; charset=UTF-8",
    },
  };

  let res = "";
  await axios
    .get(url, config)
    .then((response) => {
      isDataWriten.value = 1;
      res = response;
    })
    .catch((err) => {
      // isDataWriten.value = 2
      // requestErrorInfo.value = err
    });

  console.log("res", res);
  // if (res.status === 200)
  //   return res.data.result
  // else
  //   return "ERROR"
};

/**
 * 查找未创建的字段名称。
 * 
 * 此异步函数遍历已选中的字段名称，检查这些名称是否存在于给定的列表中。
 * 如果某个字段名称在列表中没有找到匹配项，则认为该字段尚未创建，并将其添加到结果数组中。
 * 函数最终返回一个包含所有未创建字段名称的数组。
 * 
 * @param {Array} list - 包含字段信息的列表，用于对比检查。
 * @returns {Array} 一个数组，包含所有未在列表中找到对应项的字段名称。
 */
const findUncreatedField = async (list) => {
  // 初始化一个空数组，用于收集未创建的字段名称
  const uncreatedField = [];
  
  // 遍历已选中的字段名称
  for (let name of checkedbill.value) {
    // 初始化标志变量，用于记录当前字段是否在列表中找到
    var i = 0;
    
    // 遍历给定列表，查找匹配的字段
    for (let item of list) {
      // 通过模板字符串和`t`函数构造字段的本地化名称，并打印出来
      var x = `${t(`${`info`}.${name}`)}`;
      console.log(x);
      
      // 如果找到名称匹配的字段，设置标志变量为1
      if (x == item.name) {
        i = 1;
      }
    }
    
    // 如果遍历完列表仍未找到匹配项，将该字段名称添加到未创建字段数组中
    if (i == 0) {
      uncreatedField.push(name);
    }
  }
  
  // 返回包含所有未创建字段名称的数组
  return uncreatedField;
};

/**
 * 根据名称查找选中字段的ID。
 * 
 * 该异步函数通过遍历给定列表，查找与已选中字段名称匹配的项，并返回这些字段的ID对象。
 * 主要用于在具有多个字段选项的情况下，根据用户的选择，映射出对应字段的ID，以便于后续的操作或查询。
 * 
 * @param {Array} list - 包含字段名称和ID的列表。
 * @returns {Object} 返回一个对象，其中键是字段名称，值是对应的字段ID。
 */
const findSelectedFieldIDByName = async (list) => {
  // 初始化一个空对象用于存储字段名称和对应的ID
  var fieldID = {};
  
  // 遍历已选中的字段名称
  for (let name of checkedbill.value) {
    // 遍历给定的列表，查找匹配的字段
    for (let item of list) {
      // 通过模板字符串和`t`函数构造字段的本地化名称
      var x = `${t(`${`info`}.${name}`)}`;
      // 如果找到名称匹配的字段
      if (x == item.name) {
        // 将字段名称和对应的ID存储到fieldID对象中
        fieldID[name] = item.id;
      }
    }
  }
  // 返回包含字段名称和ID的对象
  return fieldID;
};

const getData = async () => {
  if (checkedbill.value.length == 0) {
    await bitable.ui.showToast({
      toastType: "error",
      message: t("infoTip.null_error"),
    });
    return 0;
  }

  showProcessTip("start");
  const table = await bitable.base.getActiveTable();
  const fieldMetaList = await table.getFieldMetaList();
  var uncreatedField = [];
  uncreatedField = await findUncreatedField(fieldMetaList);
  await console.log("list", fieldMetaList);

  // for (var i in fieldList){
  //   table.deleteField(i.id)
  // }
  var createdField = {};
  console.log(uncreatedField)
  for (let name of uncreatedField) {
    switch (name) {
      case "Name":
        createdField[name] = await table.addField({
          type: FieldType.Text,
          name: `${t(`info.Name`)}`,
        });
        break
      case "BillID":
        await table.addField({
          type: FieldType.Text,
          name: `${t(`info.BillID`)}`,
        });
        break
      case "Price":
        await table.addField({
          type: FieldType.Currency,
          name: `${t(`info.Price`)}`,
        });
        break
      case "Time":
        await table.addField({
          type: FieldType.DateTime,
          name: `${t(`info.Time`)}`,
        });
        break
      case "invoice_link":
        await table.addField({
          type: FieldType.Url,
          name: `${t(`info.invoice_link`)}`,
        });
        break
      case "product_link":
        await table.addField({
          type: FieldType.Url,
          name: `${t(`info.product_link`)}`,
        });
        break
    }
  }

  var data = {};
  var url = `https://feishu-get-jd-order-data-wuyi.replit.app/get_jd_order_data`;
  let res;

  if (1) {
    var config = {
      method: "post",
      url: url,
      data: data,
    };

    await axios(config)
      .then(function (response) {
        console.log("jd_data >> response.data || res", response.data);
        res = response.data;
      })
      .catch(function (error) {
        console.log(error);
      });

    if (res.status === 200) {
      await writeData(table, res.data);
      

      return res.info;
    } else return { status: "-100", result: res };
  }
};

const writeData = async (table, data) => {
  const fieldMetaList = await table.getFieldMetaList();
  // console.log(data);
  var selectedFieldID = await findSelectedFieldIDByName(fieldMetaList);
  var records = [];
  for (let bill of data) {
    var record = { fields: {} };
    for (const key in selectedFieldID) {
      switch (key) {
        case "BillID":
          record["fields"][selectedFieldID[key]] = bill["order_id"];
          break
        case "Price":
          record["fields"][selectedFieldID[key]] = bill["amount"];
          break
        case "Time":
          record["fields"][selectedFieldID[key]] = bill["timestamp"];
          break
        case "invoice_link":
          record["fields"][selectedFieldID[key]] = bill["invoice_link"];
          break
        case "product_link":
          record["fields"][selectedFieldID[key]] = bill["product_link"];
          break
      }
    }
    records.push(record)
  }
  const res = await table.addRecords(records)
  console.log(res)
  showProcessTip("end");
};
// -- 辅助算法区域
const isProgressStarted = ref(false);
const isProgressEnded = ref(false);
const progressPercentage = ref(1);
const processResultDesc = ref("");

const showProcessTip = async (Tiptype) => {
  if (Tiptype === "start") {
    progressPercentage.value = 1;
    isProgressStarted.value = true;
    isProgressEnded.value = false;

    const interval = setInterval(() => {
      // 随机增加进度，模拟加载过程
      progressPercentage.value += Math.floor(Math.random() * 10);
      if (progressPercentage.value > 90) {
        clearInterval(interval);
      }
    }, 1000); // 每秒更新一次进度

    await bitable.ui.showToast({
      toastType: "loading",
      message: t("infoTip.start"),
    });
  } else if (Tiptype === "end") {
    progressPercentage.value = 100;
    isProgressStarted.value = false;
    isProgressEnded.value = true;

    await bitable.ui.showToast({
      toastType: "success",
      message: t("infoTip.end"),
    });
  }
};

onMounted(async () => {
  // 获取字段列表 -- start
  const selection = await bitable.base.getSelection();
  const table = await bitable.base.getTableById(selection.tableId);
  const view = await table.getViewById(selection.viewId);
  // 获取字段列表 -- end

  // 从缓存中获取数据 -- start
  // if (localStorage.getItem('ossConfig') !== null) {
  //   ossConfig.value = JSON.parse(localStorage.getItem('ossConfig'))
  // }
  // if (localStorage.getItem('attchImgFieldId') !== null) {
  //   attchImgFieldId.value = localStorage.getItem('attchImgFieldId')
  // }
  // if (localStorage.getItem('linkFieldId') !== null) {
  //   linkFieldId.value = localStorage.getItem('linkFieldId')
  // }
  // 从缓存中获取数据 -- end
});
</script>


<style scoped>
.time {
  margin-right: 1vw;
}
</style>