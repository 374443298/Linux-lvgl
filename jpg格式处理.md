### 上传并显示jpg格式图片
#### lvgl原生库
```
  //lvgl原生解码并显示jpg图片，但是不支持缩放
    lv_extra_init();     //extra lib init, including jpg lib init
  //lv_split_jpeg_init();
    lv_obj_t *img = lv_img_create(ui_settings3);     // creat img
    lv_img_set_src(img,"A:2.jpg");
  //  lv_img_set_zoom(img,128);
    lv_obj_move_foreground(img);
    lv_obj_center(img);
    lv_obj_align(img,LV_ALIGN_TOP_MID,0,0);
```
lvgl**8**之后就集成了文件系统的使用，只需要打开对应的宏  
https://www.cnblogs.com/jzcn/p/17045477.html  
需要注意，在lv_conf.h下，还要修改分配cache和buffer_size的大小，否则显示会乱  
这种情况下，lvgl8不支持采用lv_img_set_zoom的方式进行缩放

#### 采用stb.image  
从github上拉取这三个文件
```
#define STB_IMAGE_IMPLEMENTATION
#include "stb_image.h"
#define STB_IMAGE_WRITE_IMPLEMENTATION
#include "stb_image_write.h"
#define STB_IMAGE_RESIZE_IMPLEMENTATION
#include "stb_image_resize2.h"
```
读jpg并显示，思路是  
读.jpg到data指针，用stbi_load()读为rgb格式存储  
```
unsigned char *resized = malloc(new_width * new_height * 3);
stbir_resize_uint8_linear(data, img_width, img_height, 0, resized, new_width, new_height,0 ,3);
```
resize图片的大小，完成缩放功能，  
最后将图片转成rgba形式，手动补全alpha的值，填装lv_img_dsc_t结构体，进行显示

