#MUI #V8

# 代码示例

```jsx
<DataGrid

rows={innerRows}

columns={mockAddMoreCol}

initialState={{ pagination: { paginationModel } }} //分页配置

sx={{ border: 0 }}

checkboxSelection // 开启checkbox

disableRowSelectionOnClick // 只有选中checkbox才触发选中状态，通常开启为最佳实践

rowSelectionModel={selectModel} //行被选择的状态保存

onRowSelectionModelChange={handSelectRows} // 行被checkBox选择时调用

/>

</Paper>
```

## 重要功能

### 获取checkBox选择的行数据

1.设置一个状态去获取DataGrid组件选择的ids,handSelectRows如上方式传入给组件
```jsx
const [selectModel, setSelectModel] = React.useState({

type: "include",

ids: new Set(),

});
const handSelectRows = (newSelectModel) => {

setSelectModel(newSelectModel);

};
```

 2.当用户点击按钮时去获取selectModel.ids,去哪整个rows里面的id属性和ids做一遍过滤,getSelectRowsData就是被选择的row信息
 ```jsx
const getSelectRowsData = () => {

return innerRows.filter((row) => {

return selectModel.ids.has(row.id);

});
};
 ```

### 陷进

#### 1.在MUIX 8版本之后selectModel 从一个数组变成了一个对象，使用空数组去初始化值会导致报错

#### 2.使用checkBox,列属性一定要有一个名字为id的属性，因为这个属性会被checkBox拿去判断有哪一行被选择了

