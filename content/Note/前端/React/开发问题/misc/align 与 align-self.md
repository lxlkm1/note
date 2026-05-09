``` jsx
import { Box, TextField, Divider, Typography } from "@mui/material";
{/* align 属性是grid布局里面有的吗为什么不使用align-self */}

<Typography component={"h1"} variant="h2" align="left">

{conf.title}

</Typography>
```

# 解释

align 是 MUI  Typography 组件特有的，不是原生CSS 样式， 效果接近原生CSS text-align 属性，用来控制字体的位置

self-align 是 flex/grid 布局才有的，控制块级元素的位置