| 修饰词     | 本类 | 同一个包的类 | 继承类 | 其他类 |
| ---------- | ---- | ------------ | ------ | ------ |
| private    | √    | ×            | ×      | ×      |
| 无（默认） | √    | √            | ×      | ×      |
| protected  | √    | √            | √      | ×      |
| public     | √    | √            | √      | √      |

```
static一般用来修饰成员变量或函数。但有一种特殊用法是用static修饰内部类，普通类是不允许声明为静态的，只有内部类才可以。

被static修饰的内部类可以直接作为一个普通类来使用，而不需实例化一个外部类。

当一个内部类没有使用static修饰的时候，不能直接使用内部类创建对象，必须先使用外部类对象点new内部类对象（外部类对象.new 内部类())。
```

```java
Integer的hashcode方法
    public static int hashCode(int value) {
            return value;
    }
String的hashcode方法
    public static int hashCode(byte[] value) {//for StringLatin1
        int h = 0;
        for (byte v : value) {
            h = 31 * h + (v & 0xff);//8位的byte，直接以31为基数求余数的逆运算
        }
        return h;
    }
	public static int hashCode(byte[] value) {//for StringUTF16
        int h = 0;
        int length = value.length >> 1;
        for (int i = 0; i < length; i++) {
            h = 31 * h + getChar(value, i);//8位的byte，首先左移8位再加上原来的byte，变为16位数再以31为基数求余数的逆运算
        }
        return h;
    }
    static char getChar(byte[] val, int index) {
        assert index >= 0 && index < length(val) : "Trusted caller missed bounds check";
        index <<= 1;
        return (char)(((val[index++] & 0xff) << HI_BYTE_SHIFT) |
                      ((val[index]   & 0xff) << LO_BYTE_SHIFT));
    }
    private static native boolean isBigEndian();

    static final int HI_BYTE_SHIFT;
    static final int LO_BYTE_SHIFT;
    static {
        if (isBigEndian()) {//Java为大端模式，即高位字节存储在低位地址上
            HI_BYTE_SHIFT = 8;
            LO_BYTE_SHIFT = 0;
        } else {
            HI_BYTE_SHIFT = 0;
            LO_BYTE_SHIFT = 8;
        }
    }
```

