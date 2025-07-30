```
#ifndef KEYBOARD_H
#define KEYBOARD_H

#include "lvgl.h"

#ifdef __cplusplus
extern "C" {
#endif

// 自定义键盘结构体
typedef struct {
    lv_obj_t* container;        // 键盘主容器
    lv_obj_t* text_area;        // 输入文本框
    lv_obj_t* buttons[32];      // 按键数组 (32个按键)
    bool is_visible;            // 是否可见
} custom_keyboard_t;

// 函数声明
custom_keyboard_t* keyboard_create(lv_obj_t* parent);
void keyboard_destroy(custom_keyboard_t* keyboard);
void keyboard_show(custom_keyboard_t* keyboard);
void keyboard_hide(custom_keyboard_t* keyboard);
bool keyboard_is_visible(custom_keyboard_t* keyboard);

#ifdef __cplusplus
}
#endif

#endif // KEYBOARD_H 
```

```
#include "keyboard.h"
#include <stdlib.h>
#include <string.h>

// 键盘布局定义 - 根据图片重新设计
static const char* key_labels[] = {
    // 第一行：q, w, e, r, t, y, u, i, o, p
    "q", "w", "e", "r", "t", "y", "u", "i", "o", "p",
    // 第二行：Shift, a, s, d, f, g, h, j, k, l
    "Shift", "a", "s", "d", "f", "g", "h", "j", "k", "l",
    // 第三行：123, z, x, c, v, b, n, m, Backspace
    "123", "z", "x", "c", "v", "b", "n", "m", "Backspace",
    // 第四行：BACK, SPACE, OK
    "BACK", "SPACE", "OK"
};

// 按键位置和大小定义
static const struct {
    int x, y, width, height;
} key_positions[] = {
    // 第一行：q, w, e, r, t, y, u, i, o, p
    {5, 65, 64, 64}, {74, 65, 64, 64}, {143, 65, 64, 64}, {212, 65, 64, 64}, {281, 65, 64, 64},
    {350, 65, 64, 64}, {419, 65, 64, 64}, {488, 65, 64, 64}, {557, 65, 64, 64}, {626, 65, 64, 64},

    // 第二行：Shift, a, s, d, f, g, h, j, k, l
    {5, 137, 64, 64}, {74, 137, 64, 64}, {143, 137, 64, 64}, {212, 137, 64, 64}, {281, 137, 64, 64},
    {350, 137, 64, 64}, {419, 137, 64, 64}, {488, 137, 64, 64}, {557, 137, 64, 64}, {626, 137, 64, 64},

    // 第三行：123, z, x, c, v, b, n, m, Backspace
    {5, 209, 97, 64},   // 123
    {107, 209, 64, 64},  // z
    {176, 209, 64, 64},  // x
    {245, 209, 64, 64},  // c
    {314, 209, 64, 64},  // v
    {383, 209, 64, 64},  // b
    {452, 209, 64, 64},  // n
    {521, 209, 64, 64},  // m
    {590, 209, 101, 64}, // Backspace

    // 第四行：BACK, SPACE, OK
    {5, 281, 97, 64},    // BACK
    {107, 281, 408, 64},  // SPACE
    {520, 281, 171, 64}   // OK
};

// 按钮点击事件回调
static void keyboard_button_event_cb(lv_event_t* e)
{
    lv_obj_t* btn = lv_event_get_target(e);
    lv_event_code_t code = lv_event_get_code(e);
    
    if (code == LV_EVENT_CLICKED) {
        const char* text = lv_label_get_text(lv_obj_get_child(btn, 0));
        custom_keyboard_t* keyboard = (custom_keyboard_t*)lv_obj_get_user_data(btn);
        
        if (keyboard && keyboard->text_area) {
            if (strcmp(text, "Backspace") == 0) {
                // 退格键
                lv_textarea_del_char(keyboard->text_area);
            } else if (strcmp(text, "SPACE") == 0) {
                // 空格键
                lv_textarea_add_char(keyboard->text_area, ' ');
            } else if (strcmp(text, "OK") == 0) {
                // 确认键
                keyboard_hide(keyboard);
            } else if (strcmp(text, "BACK") == 0) {
                // 返回键
                keyboard_hide(keyboard);
            } else if (strcmp(text, "123") == 0) {
                // 数字模式切换（这里可以扩展）
                // TODO: 实现数字模式
            } else if (strcmp(text, "Shift") == 0) {
                // 大小写切换（这里可以扩展）
                // TODO: 实现大小写切换
            } else {
                // 普通字符
                lv_textarea_add_char(keyboard->text_area, text[0]);
            }
        }
    } else if (code == LV_EVENT_PRESSED) {
        // 按下时改变颜色
        lv_obj_set_style_bg_color(btn, lv_color_hex(0xA1A1A1), 0);
        lv_obj_set_style_text_color(btn, lv_color_hex(0xFF520F), 0);
    } else if (code == LV_EVENT_RELEASED) {
        // 释放时恢复颜色
        lv_obj_set_style_bg_color(btn, lv_color_hex(0xA1A1A1), 0);
        lv_obj_set_style_text_color(btn, lv_color_hex(0xFFFFFF), 0);
    }
}

// 创建键盘
custom_keyboard_t* keyboard_create(lv_obj_t* parent)
{
    custom_keyboard_t* keyboard = malloc(sizeof(custom_keyboard_t));
    if (!keyboard) return NULL;
    
    // 初始化键盘对象
    memset(keyboard, 0, sizeof(custom_keyboard_t));
    keyboard->is_visible = false;
    
    // 创建键盘主容器
    keyboard->container = lv_obj_create(parent);
    lv_obj_set_size(keyboard->container, 696, 357);
    lv_obj_set_style_bg_color(keyboard->container, lv_color_hex(0x2A2A2A), 0);
    lv_obj_set_style_border_width(keyboard->container, 0, 0);
    lv_obj_set_style_pad_all(keyboard->container, 0, 0);
    lv_obj_set_style_radius(keyboard->container, 0, 0);
    lv_obj_clear_flag(keyboard->container, LV_OBJ_FLAG_SCROLLABLE); // 禁止滚动
    
    // 创建文本框
    keyboard->text_area = lv_textarea_create(keyboard->container);
    lv_obj_set_size(keyboard->text_area, 685, 44);
    lv_obj_set_pos(keyboard->text_area, 13, 13);
    lv_obj_set_style_bg_color(keyboard->text_area, lv_color_hex(0xFFFFFF), 0);
    lv_obj_set_style_border_width(keyboard->text_area, 0, 0);
    lv_obj_set_style_radius(keyboard->text_area, 0, 0);
    lv_obj_set_style_text_color(keyboard->text_area, lv_color_hex(0x000000), 0);
    lv_obj_set_style_pad_all(keyboard->text_area, 10, 0);
    
    // 创建按键
    for (int i = 0; i < 32; i++) {
        keyboard->buttons[i] = lv_btn_create(keyboard->container);
        lv_obj_set_size(keyboard->buttons[i], key_positions[i].width, key_positions[i].height);
        lv_obj_set_pos(keyboard->buttons[i], key_positions[i].x, key_positions[i].y);
        
        // 设置按键样式
        lv_obj_set_style_bg_color(keyboard->buttons[i], lv_color_hex(0xA1A1A1), 0);
        lv_obj_set_style_bg_opa(keyboard->buttons[i], LV_OPA_50, 0); // 设置不透明度为45%
        lv_obj_set_style_text_color(keyboard->buttons[i], lv_color_hex(0xFFFFFF), 0);
        lv_obj_set_style_radius(keyboard->buttons[i], 6, 0);
        lv_obj_set_style_border_width(keyboard->buttons[i], 0, 0);
        lv_obj_set_style_pad_all(keyboard->buttons[i], 5, 0);
        
        // 创建标签
        lv_obj_t* label = lv_label_create(keyboard->buttons[i]);
        lv_label_set_text(label, key_labels[i]);
        lv_obj_center(label);
        lv_obj_set_style_text_font(label, &lv_font_montserrat_24, 0); // 字号24px
        
        // 设置用户数据为键盘对象
        lv_obj_set_user_data(keyboard->buttons[i], keyboard);
        
        // 添加事件回调
        lv_obj_add_event_cb(keyboard->buttons[i], keyboard_button_event_cb, LV_EVENT_CLICKED, NULL);
        lv_obj_add_event_cb(keyboard->buttons[i], keyboard_button_event_cb, LV_EVENT_PRESSED, NULL);
        lv_obj_add_event_cb(keyboard->buttons[i], keyboard_button_event_cb, LV_EVENT_RELEASED, NULL);
    }
    
    // 初始隐藏键盘
    lv_obj_add_flag(keyboard->container, LV_OBJ_FLAG_HIDDEN);
    
    return keyboard;
}

// 销毁键盘
void keyboard_destroy(custom_keyboard_t* keyboard)
{
    if (keyboard) {
        if (keyboard->container) {
            lv_obj_del(keyboard->container);
        }
        free(keyboard);
    }
}

// 显示键盘
void keyboard_show(custom_keyboard_t* keyboard)
{
    if (keyboard && keyboard->container) {
        lv_obj_clear_flag(keyboard->container, LV_OBJ_FLAG_HIDDEN);
        keyboard->is_visible = true;
    }
}

// 隐藏键盘
void keyboard_hide(custom_keyboard_t* keyboard)
{
    if (keyboard && keyboard->container) {
        lv_obj_add_flag(keyboard->container, LV_OBJ_FLAG_HIDDEN);
        keyboard->is_visible = false;
    }
}

// 检查键盘是否可见
bool keyboard_is_visible(custom_keyboard_t* keyboard)
{
    return keyboard ? keyboard->is_visible : false;
}

```
