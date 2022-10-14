# webgl


canvas 的坐标系统，以原点在左上方，Y轴正方向下，X轴正方朝右

## 基础方法
```javascript
/**
 * 指定绘图区域
 * @param red {float} 0.0-1.0
 * @param green {float} 0.0-1.0
 * @param blue {float} 0.0-1.0
 * @param alpha {float} 0.0-1.0
 * @return null
 */
gl.clear(red,green,blue,alpha)
```

```javascript
/**
 * 将指定缓冲区设定为预定的值。如果清空的是颜色缓冲区，那么将使用gl.clearColor()指定值（作为预设值）
 * @param buffer [
 *   COCOLR_BUFFER_BIT  指定颜色缓存
 *   DEPTH_BUFFER_BIT   指定深度缓冲区
 *   STENCIL_BUFFER_BIT 指定模版缓冲区
 * ]
 */
gl.clear(buffer);

gl.clearColor(red,green,blue, alpha);
gl.clearDepth(depth);
gl.clearStencil(s);
```

## 着色器
坐标原点在canvas中心，使用三维坐标系（笛卡尔坐标系）

### 着色器的加载处理

### 代码展示

```
// 顶点着色器
void main() {
    gl_Position = vec4(0.0, 0.0, 0.0, 1.0); //齐次坐标
    gl_PointSize = 10.0;
}

// 片元着色器
void main() {
    gl_FragColor = vec4(1.0, 0.0, 0.0, 1.0) //RGBA格式
}

```
`
齐次坐标
齐次坐标使用如(x,y,z,w)描述,到canvas边框距离为1，等价于三维坐标(x/w,y/w,z/w),如w为0，则表示坐标无穷远
`

### js和shader基本通讯

定点着色器交互
```
// 配置定点着色器
attribute vec4 a_Position;
void main() {
    gl_Postion = a_Positon;
    gl_PointSize = 10.0;
}
```

```javascript
/**
 * getAttribLocation
 * @param program {WebGLProgram} 指定包含定点着色器和片元着色器程序对象
 * @param name {string} 指定想要获取其他存储地址的attribute变量的名称
 * @return 变量位置
 */
var a_Position = gl.getAttribLocation(gl.program, 'a_Position');
//注入相关节点，设置值
gl.vertexAttrib3fv(a_Position, 0.0, 0.0, 0.0);
//进行绘制
/**
 * drawArrays
 * @param mode {gl.POINTS | gl.LINES | gl.LINES_STRIP | gl.LINES_LOOP | gl.TRIANGLES | gl.TRIANGLE_STRIP | gl.TRIANGLE_FAN}
 * @param first INT 从哪个顶点开始绘制
 * @param count INT 需要绘制多少个顶点
 * 
 */
gl.drawArrays(gl.POINTS, 0, 1);

// 另外可以这样写
var positions = new Float32Array([1.0, 2.0, 3.0, 1.0]);
gl.vertexAttrib4fv(positions);

// 还可以这样写,有这些的同族函数
gl.vertexAttrib1fv(location, v0);
gl.vertexAttrib2fv(location, v0, v1);
gl.vertexAttrib3fv(location, v0, v1, v2);
gl.vertexAttrib4fv(location, v0, v1, v2, v3);
```
片元着色器交互

```
precision mediump float;
uniform vec4 u_FragColor;
void main() {
    gl_FragColor = u_FragColor;
}
```

```javascript
/**
 * getUniformLocation
 * @param program {WebGLProgram} 指定包含定点着色器和片元着色器程序对象
 * @param name {string}  指定想获取其他存储位置的uniform变量名称
 * @return 变量位置
 */
var u_FragColor = gl.getUniformLocation(gl.program, 'u_FragColor');

gl.uniform4f(u_FragColor, 1.0, 0.0, 0.0 , 1.0);
gl.drawArrays(gl.POINTS, 0, 1);
```
webgl创建缓冲对象的基本流程(创建数据)
```javascript
//1.创建缓冲区对象
/**
 * createBuffer
 * @return  WebGLBuffer
 */
gl.createBuffer();

// 删除缓冲区对象
/**
 * deleteBuffer
 * @param buffer {WebGLBuffer}
 */
gl.deleteBuffer(buffer)

//2.绑定缓冲对象
/**
 * bindBuffer
 * @param target {gl.ARRAY_BUFFER | gl.ELEMENT_ARRAY_BUFFER}
 * ARRAY_BUFFER: 包含了顶点的数据
 * ELEMENT_ARRAY_BUFFER: 包含了顶点的索引值
 * @param buffer 待绑定的缓冲区对象
 */
gl.bindBuffer(target, buffer);

// 3.将数据写入缓冲区对象
/**
 * bufferData
 * @param target {gl.ARRAY_BUFFER | gl.ELEMENT_ARRAY_BUFFER} 
 * @param data 类型化数据 如，new Float32Array()
 * @param usage {gl.STATIC_DRAW | gl.STREAM_DRAW | gl.DYNAMIC_DRAW}  如何使用存储在缓冲区对象中的数据
 * STATIC_DRAW 写入一次数据，需要绘制很多次
 * STREAM_DRAW 写入一次数据，然后绘制若干次
 * DYNAMIC_DRAW 多次写入数据，然后绘制很多次
 */
gl.bufferData(target, data, usage);

// 4.将缓冲区对象分配给一个attribute变量
/**
 * vertexAttribPointer
 * @param location attribute变量的存储位置
 * @param size 缓冲区每一个顶点的个数(1-4),有点像vertexAttribute[1 2 3 4]f();
 * @param type 定义数据类型 {gl.UNSIGNED_BYTE | gl.SHOT | gl.UNSIGNED_SHOT | gl.INT | gl.UNSIGNED_INT | gl.FLOAT}
 * UNSIGNED_BYTE 无符号字节
 * SHOT 短整型
 * UNSIGNED_SHOT 无符号短整型
 * INT 整型
 * UNSIGNED_INT 无符号整型
 * FLOAT 浮点型
 * @param normalize {boolean} 是否把非浮点型的数据规划[0,1]或[-1, 1]之间
 * @param strde 相邻两个顶点的字节数
 * @param offset 从缓冲区中何处开始存值（以字节为单位）
 */
gl.vertexAttribPointer(location, size, type, normalize,stride, offset);

// 5.开启attribute变量
/**
 * enableVertexAttribArray
 * @param location 指定attribue变量的存储位置
 */
gl.enableVertexAttribArray(location);

/**
 * disableVertexAttribArray
 * 关闭分配
 * @param location 指定attribute变量的存储位置
 */
gl.disableVertexAttribArray(location)
```

webgl与JavaScript点击
```javascript
function click(e) {
    var x = e.clientX;
    var y = e.clientY;
    var rect = e.target.getBoundingClientRect();
    x = ((x - rect.left) - canvas.height/2)/(canvas.height/2);
    y = (canvas.width/2 - (y - rect.top))/(canvas.width/2);
    
}
```

# canvas 2d

相关资料介绍
https://test.domojyun.net/MEMO/Canvas/
