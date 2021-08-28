# 避免switch嵌套的一种方法

```cpp
DWORD nFlags = 0;
		switch (mouse.nButton)
		{
		case 0: //左键
			nFlags = 1;

		case 1: //右键
			nFlags = 2;
			break;
		case 2: //中间
			nFlags = 4;
			break;
		case 4:
			nFlags = 8;
			break;
		}
		if (nFlags != 8)
			SetCursorPos(mouse.ptXY.x, mouse.ptXY.y);
		switch (mouse.nAction)
		{
		case 0: //单击
			nFlags |= 0x10;
			break;
		case 1: //双击
			nFlags |= 0x20;
			break;
		case 2: //按下
			nFlags |= 0x40;
			break;
		case 3: //放开
			nFlags |= 0x80;
			break;

		default:
			break;
		}
		switch (nFlags)
		{
		case 0x21: //左键双击
		case 0x11: //左键单击
			break;

		case 0x41: //左键按下
			
			break;
		case 0x81: //左键放开
			
			break;
		case 0x22:
			
		case 0x12:
			break;
		case 0x42:
			break;
		case 0x82:
			break;
		case 0x24:
		case 0x14:
			break;
		case 0x44:
			break;
        case 0x84:
			break;
		case 0x08:
			break;
		}
```

在第一个`switch`中设置低位然后再下一个`switch`设置高位,最后根据不同情况进行选择