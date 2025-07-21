### 上传并显示jpg格式图片

lv_obj_t *objpg = lv_img_create(main_scr, NULL); // 创建一个IMG对象
lv_img_set_src(objpg, "S:/image/test_jpg.jpg"); // 加载SD卡中的JPG图片
lv_obj_align(objpg, NULL, LV_ALIGN_IN_TOP_LEFT, 0, 0); // 重新设置对齐  

