```c
#include "../include/global.h"
#include "../include/stdint.h"
#include "../include/io.h"
#include "../include/console.h"
#include "../include/keyboard.h"



/** KEYBORD */
#define PORT_KBD_BUF 0x60

/** CHAR and KEY */
#define CHAR_NUL 0
#define CHAR_ESC '\033'
#define CHAR_TAB '\t'
#define CHAR_BACKSPACE '\b'
#define CHAR_DELETE '\177'
#define CHAR_ENTER '\n'

#define KEY_CTRL_L 0x1d
#define KEY_CTRL_R 0xe01d
#define KEY_SHIFT_L 0x2a
#define KEY_SHIFT_R 0x36
#define KEY_ALT_L 0x38
#define KEY_ALT_R 0xe038

bool is_mulit_key,is_break;
bool ctrl_status,shift_status,alt_status,ctrl_status_last,shift_status_last,alt_status_last;

static char keymap[][3] = {
    {0      ,0},                /** 0x00 **/
    {CHAR_ESC    ,CHAR_ESC},    /** 0x01 **/
    {'1'    ,'!'},              /** 0x02 **/
    {'2'    ,'@'},              /** 0x03 **/
    {'3'    ,'#'},              /** 0x04 **/
    {'4'    ,'$'},               /** 0x05 **/
    {'5'    ,'%'},               /** 0x06**/
    {'6'    ,'^'},              /** 0x07**/
    {'7'    ,'&'},              /** 0x08**/
    {'8'    ,'*'},              /** 0x09**/
    {'9'    ,'('},              /** 0x0A**/
    {'0'    ,')'},              /** 0x0B**/
    {'-'    ,'_'},              /** 0x0C**/
    {'='    ,'+'},              /** 0x0D**/
    {CHAR_BACKSPACE    ,CHAR_BACKSPACE}, /** 0x0E**/
    {CHAR_TAB          ,CHAR_TAB},       /** 0x0F**/   
    {'q'    ,'Q'},              /** 0x10 **/
    {'w'    ,'W'},              /** 0x11 **/
    {'e'    ,'E'},              /** 0x12 **/
    {'r'    ,'R'},              /** 0x13 **/
    {'t'    ,'T'},              /** 0x14 **/
    {'y'    ,'Y'},              /** 0x15 **/
    {'u'    ,'U'},              /** 0x16 **/
    {'i'    ,'I'},              /** 0x17 **/
    {'o'    ,'O'},              /** 0x18 **/
    {'p'    ,'P'},              /** 0x19 **/
    {'['    ,'{'},              /** 0x1A **/
    {']'    ,'}'},              /** 0x1B**/
    {CHAR_ENTER    ,CHAR_ENTER},              /** 0x1C **/
    {CHAR_NUL      ,CHAR_NUL},              /** 0x1D **/
    {'a'    ,'A'},              /** 0x1E **/
    {'s'    ,'S'},              /** 0x1F **/
    {'d'    ,'D'},              /** 0x20 **/
    {'f'    ,'F'},              /** 0x21 **/
    {'g'    ,'G'},              /** 0x22 **/
    {'h'    ,'H'},              /** 0x23 **/
    {'j'    ,'J'},              /** 0x24 **/
    {'k'    ,'K'},              /** 0x25 **/
    {'l'    ,'L'},              /** 0x26 **/
};


void key_down(uint8 scancode){

    //读取多扫描码按键
    if(scancode == 0xe0) {
        is_mulit_key = true;
        return;
    }
    if(is_mulit_key) {
        scancode =  0xe000 | scancode;
        is_mulit_key = false;
    }

    //判断是否为断码：第8位为1是断码
   is_break = (scancode & 0x0080) != 0 ;
    if(is_break) {
        uint8 make_scancode =  scancode & 0xff7f;
        if(make_scancode == KEY_CTRL_L || make_scancode == KEY_CTRL_R) {
            ctrl_status = false;
        } 
        if(make_scancode == KEY_SHIFT_L || make_scancode == KEY_SHIFT_L) {
            shift_status= false;
        } 
        if(make_scancode == KEY_ALT_L || make_scancode == KEY_ALT_R ) {
            alt_status = false;
        } 
        return;

    } else {

        uint8 real_scancode =  scancode & 0x00ff;

        if(real_scancode >0x00 && real_scancode <0x3b) {
            char mapchar = keymap[real_scancode][shift_status];
            if(mapchar) {
                put_char(mapchar);
                return;
            }
        }
        
        
        if(real_scancode == KEY_CTRL_L || real_scancode == KEY_CTRL_R) {
            ctrl_status = true;
        } 
        if(real_scancode == KEY_SHIFT_L || real_scancode == KEY_SHIFT_L) {
            shift_status= true;
        } 
        if(real_scancode == KEY_ALT_L || real_scancode == KEY_ALT_R ) {
            alt_status = true;
        } 

        
    }
    
}


/** 
 *  按键回调函数，由io_keyboard_handler回调
 *
 */
void keyboard_handler_callback()
{

     //读取键盘缓冲器
    uint8 scancode = io_in8(PORT_KBD_BUF);

    key_down( scancode);

    //EOI结束中断,以便开始下次中断
    io_out8(PIC0_ICW1,0x20);

    //putstring(" do keyboard_handler_callback!\n");
}




void init_keyboard() {
    register_handler(0x21, io_keyboard_handler);
    put_string("init_keyboard!\n");
}
```

