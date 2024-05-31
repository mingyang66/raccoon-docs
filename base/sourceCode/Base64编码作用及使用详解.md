### Base64编码作用及使用详解

```
介绍：Base64是网络上最常见的用于传输8Bit字节码的编码方式之一，Base64就是一种基于64个可打印字符来表示二进制数据的方法；
```

##### 1.使用场景：

```
Base64一般用于在HTTP协议下传输二进制数据，由于HTTP协议是文本协议，所以在HTTP协议下传输二进制数据需要将二进制数据转换字符数据；然而直接转换是不行的，因为网络传输只能传输可打印字符；
什么是可打印字符？在ASCII码中规定，0-31、127这33个字符属于控制字符，32-126这95个字符属于可打印字符，也就是说网络传输只能传输这95个字符，不在这个范围内的字符无法传输。那么该怎么才能传输其它字符呢？其中一种方式就是使用Base64
```

##### 2.Base64编码分类

- Basic编码，输出被映射到一组字符A-Za-z0-9+/,编码不添加任何航标，输出的解码仅支持A-Za-z0-9+/
- URL编码：输出映射到一组字符A-Za-z0-9+_，输出的是URL和文件
- MINE编码：使用Basic编码，输出映射到MIME(Multipurpose Internet Mail Extensions-多用途互联网邮件扩展类型)
  友好格式，输出每行不超过76个字符，并使用'\r\n'作为分隔符；

>
Basic编码将指定的数据转换为base64编码，其中会将问号转换为斜杠，如：“runoob?java8”中的问号转换为“/”;URL编码和Basic编码的不同之处是会将问号转换为下横杠，如：“runoob?java8”中的问号转换为“_
”，主要用在url参数传递时的编码和解码;那MINE编码呢？它是基于Basic编码，但是会输出格式友好的数据；

示例可以参考：

[https://www.runoob.com/java/java8-base64.html]( https://www.runoob.com/java/java8-base64.html)

##### 3.自己封装的工具类

```java
package com.emily.boot.common.utils.hash;

import com.emily.boot.common.enums.AppHttpStatus;
import com.emily.boot.common.exception.BusinessException;
import com.emily.boot.common.utils.CharsetUtils;
import com.emily.boot.common.utils.io.IOUtils;
import org.apache.commons.lang3.StringUtils;

import java.io.*;
import java.util.Base64;

/**
 * 介绍：Base64是网络上最常见的用于传输8Bit字节码的编码方式之一，Base64就是一种基于64个可打印字符来表示二进制数据的方法；
 * 使用场景：Base64一般用于在HTTP协议下传输二进制数据，由于HTTP协议是文本协议，所以在HTTP协议下传输二进制数据需要将二进制数据转换字符数据；
 * 然而直接转换是不行的，因为网络传输只能传输可打印字符；
 * 什么是可打印字符？在ASCII码中规定，0-31、127这33个字符属于控制字符，32-126这95个字符属于可打印字符，也就是说网络传输只能传输这95个字符，
 * 不在这个范围内的字符无法传输。那么该怎么才能传输其它字符呢？其中一种方式就是使用Base64
 *
 * @program: spring-parent
 *  base64编码工具类
 * @create: 2020/05/21
 */
public class Base64Utils {
    /**
     * Basic编码：输出被映射到一组字符A-Za-z0-9+/,编码不添加任何航标，输出的解码仅支持A-Za-z0-9+/
     * etc:runoob?java8 其中？可以转换为/
     *
     * @param encoderStr 字符串,默认UTF-8
     * @return 编码后的字符串
     */
    public static String encoder(String encoderStr) {
        return encoder(encoderStr, CharsetUtils.UTF_8);
    }

    /**
     * 字符串编码
     * Basic编码：输出被映射到一组字符A-Za-z0-9+/,编码不添加任何航标，输出的解码仅支持A-Za-z0-9+/
     * etc:runoob?java8 其中？可以转换为/
     *
     * @param encoderStr 字符串
     * @param charset    字符编码类型
     * @return 编码后的字符串
     */
    public static String encoder(String encoderStr, String charset) {
        try {
            if (StringUtils.isEmpty(encoderStr)) {
                throw new BusinessException(AppHttpStatus.NULL_POINTER_EXCEPTION.getStatus(), "编码字符串不可以为空");
            }
            if (StringUtils.isEmpty(charset)) {
                charset = CharsetUtils.UTF_8;
            }
            return Base64.getEncoder().encodeToString(encoderStr.getBytes(charset));
        } catch (UnsupportedEncodingException e) {
            throw new BusinessException(AppHttpStatus.ENCODING_UNSUPPORTED_EXCEPTION);
        }
    }

    /**
     * 字符串解码
     * Basic编码：输出被映射到一组字符A-Za-z0-9+/,编码不添加任何航标，输出的解码仅支持A-Za-z0-9+/
     * etc:runoob?java8 其中？可以转换为/
     *
     * @param decoderStr 字符串
     * @return 解码后的字符串
     */
    public static String decoder(String decoderStr) {
        return decoder(decoderStr, CharsetUtils.UTF_8);
    }

    /**
     * 字符串解码
     * Basic编码：输出被映射到一组字符A-Za-z0-9+/,编码不添加任何航标，输出的解码仅支持A-Za-z0-9+/
     * etc:runoob?java8 其中？可以转换为/
     *
     * @param decoderStr 字符串
     * @param charset    编码方式
     * @return 解码后的字符串
     */
    public static String decoder(String decoderStr, String charset) {
        try {
            if (StringUtils.isEmpty(decoderStr)) {
                throw new BusinessException(AppHttpStatus.NULL_POINTER_EXCEPTION.getStatus(), "解码字符串不可以为空");
            }
            if (StringUtils.isEmpty(charset)) {
                charset = CharsetUtils.UTF_8;
            }
            return new String(Base64.getDecoder().decode(decoderStr), charset);
        } catch (UnsupportedEncodingException e) {
            throw new BusinessException(AppHttpStatus.ENCODING_UNSUPPORTED_EXCEPTION);
        }
    }

    /**
     * URL编码：输出映射到一组字符A-Za-z0-9+_，输出的是URL和文件
     * etc:runoob?java8 其中？可以转换为_ ;对URL参数友好
     *
     * @param encoderStr 编码字符串,默认UTF-8
     * @return 编码后的字符串
     */
    public static String urlEncoder(String encoderStr) {
        return urlEncoder(encoderStr, CharsetUtils.UTF_8);
    }

    /**
     * URL编码：输出映射到一组字符A-Za-z0-9+_，输出的是URL和文件
     * etc:runoob?java8 其中？可以转换为_ ;对URL参数友好
     *
     * @param encoderStr 编码字符串
     * @param charset    编码方式
     * @return 编码后的字符串
     */
    public static String urlEncoder(String encoderStr, String charset) {
        try {
            if (StringUtils.isEmpty(encoderStr)) {
                throw new BusinessException(AppHttpStatus.NULL_POINTER_EXCEPTION.getStatus(), "编码字符串不可以为空");
            }
            if (StringUtils.isEmpty(charset)) {
                charset = CharsetUtils.UTF_8;
            }
            return Base64.getUrlEncoder().encodeToString(encoderStr.getBytes(charset));
        } catch (UnsupportedEncodingException e) {
            throw new BusinessException(AppHttpStatus.ENCODING_UNSUPPORTED_EXCEPTION);
        }
    }

    /**
     * URL解码：输出映射到一组字符A-Za-z0-9+_，输出的是URL和文件
     * etc:runoob?java8 其中？可以转换为_ ;对URL参数友好
     *
     * @param decoderStr 解码字符串
     * @return 解码后的字符串
     */
    public static String urlDecoder(String decoderStr) {
        return urlDecoder(decoderStr, CharsetUtils.UTF_8);
    }

    /**
     * URL解码：输出映射到一组字符A-Za-z0-9+_，输出的是URL和文件
     * etc:runoob?java8 其中？可以转换为_ ;对URL参数友好
     *
     * @param decoderStr 解码字符串
     * @param charset    解码方式
     * @return 解码后的字符串
     */
    public static String urlDecoder(String decoderStr, String charset) {
        try {
            if (StringUtils.isEmpty(decoderStr)) {
                throw new BusinessException(AppHttpStatus.NULL_POINTER_EXCEPTION.getStatus(), "解码字符串不可以为空");
            }
            if (StringUtils.isEmpty(charset)) {
                charset = CharsetUtils.UTF_8;
            }
            return new String(Base64.getUrlDecoder().decode(decoderStr), charset);
        } catch (UnsupportedEncodingException e) {
            throw new BusinessException(AppHttpStatus.ENCODING_UNSUPPORTED_EXCEPTION);
        }
    }

    /**
     * MIME编码：输出映射到MIME(Multipurpose Internet Mail Extensions-多用途互联网邮件扩展类型)友好格式，输出每行不超过76个字符，并使用'\r\n'作为分隔符；
     * 编码输出最后没有行分割
     *
     * @param encoderStr 编码字符串
     * @return 编码后的字符串
     */
    public static String mineEncoder(String encoderStr) {
        return mineEncoder(encoderStr, CharsetUtils.UTF_8);
    }

    /**
     * MIME编码：输出映射到MIME(Multipurpose Internet Mail Extensions-多用途互联网邮件扩展类型)友好格式，输出每行不超过76个字符，并使用'\r\n'作为分隔符；
     * 编码输出最后没有行分割
     *
     * @param encoderStr 编码字符串
     * @param charset    编码类型
     * @return 编码后的字符串
     */
    public static String mineEncoder(String encoderStr, String charset) {
        try {
            if (StringUtils.isEmpty(encoderStr)) {
                throw new BusinessException(AppHttpStatus.NULL_POINTER_EXCEPTION.getStatus(), "编码字符串不可以为空");
            }
            if (StringUtils.isEmpty(charset)) {
                charset = CharsetUtils.UTF_8;
            }
            return Base64.getMimeEncoder().encodeToString(encoderStr.getBytes(charset));
        } catch (UnsupportedEncodingException e) {
            throw new BusinessException(AppHttpStatus.ENCODING_UNSUPPORTED_EXCEPTION);
        }
    }

    /**
     * MIME编码：使用Base编码模式，输出映射到MIME(Multipurpose Internet Mail Extensions-多用途互联网邮件扩展类型)友好格式，输出每行不超过76个字符，并使用'\r\n'作为分隔符；
     * 编码输出最后没有行分割
     *
     * @param decoderStr 解码字符串
     * @return 解码后的字符串
     */
    public static String mineDecoder(String decoderStr) {
        return mineDecoder(decoderStr, CharsetUtils.UTF_8);
    }

    /**
     * MIME编码：使用Base编码模式，输出映射到MIME(Multipurpose Internet Mail Extensions-多用途互联网邮件扩展类型)友好格式，输出每行不超过76个字符，并使用'\r\n'作为分隔符；
     * 编码输出最后没有行分割
     *
     * @param decoderStr 解码字符串
     * @param charset    字符编码
     * @return 解码后的字符串
     */
    public static String mineDecoder(String decoderStr, String charset) {
        try {
            if (StringUtils.isEmpty(decoderStr)) {
                throw new BusinessException(AppHttpStatus.NULL_POINTER_EXCEPTION.getStatus(), "解码字符串不可以为空");
            }
            if (StringUtils.isEmpty(charset)) {
                charset = CharsetUtils.UTF_8;
            }
            return new String(Base64.getMimeDecoder().decode(decoderStr), charset);
        } catch (UnsupportedEncodingException e) {
            throw new BusinessException(AppHttpStatus.ENCODING_UNSUPPORTED_EXCEPTION);
        }
    }

    /**
     * Base编码，将指定的字符串使用base64编码存入指定的文件
     *
     * @param filePath   指定文件路径
     * @param encoderStr 编码字符串
     * @param charset    字符编码
     */
    public static void encoderWrap(String filePath, String encoderStr, String charset) {
        try {
            if (StringUtils.isEmpty(charset)) {
                charset = CharsetUtils.UTF_8;
            }
            OutputStream os = Base64.getEncoder().wrap(new FileOutputStream(new File(filePath)));
            IOUtils.write(encoderStr.getBytes(charset), os);
        } catch (FileNotFoundException e) {
            throw new BusinessException(AppHttpStatus.READ_RESOURSE_NOT_FOUND_EXCEPTION.getStatus(), StringUtils.join("指定路径", filePath, "的文件不存在," + e));
        } catch (UnsupportedEncodingException e) {
            throw new BusinessException(AppHttpStatus.ENCODING_UNSUPPORTED_EXCEPTION);
        }
    }

    /**
     * Base编码，将指定文件的内容解码为字符串
     *
     * @param filePath 指定文件路径
     * @param charset  字符编码
     * @return 解码后的字符串
     */
    public static String decoderWrap(String filePath, String charset) {
        try {
            if (StringUtils.isEmpty(charset)) {
                charset = CharsetUtils.UTF_8;
            }
            InputStream is = Base64.getDecoder().wrap(new FileInputStream(new File(filePath)));
            return IOUtils.toString(is, charset);
        } catch (FileNotFoundException e) {
            throw new BusinessException(AppHttpStatus.READ_RESOURSE_NOT_FOUND_EXCEPTION.getStatus(), StringUtils.join("指定路径", filePath, "的文件不存在," + e));
        }
    }

    /**
     * URL编码，将指定的字符串使用base64编码存入指定的文件
     *
     * @param filePath   指定文件路径
     * @param encoderStr 编码字符串
     * @param charset    字符编码
     */
    public static void urlEncoderWrap(String filePath, String encoderStr, String charset) {
        try {
            if (StringUtils.isEmpty(charset)) {
                charset = CharsetUtils.UTF_8;
            }
            OutputStream os = Base64.getUrlEncoder().wrap(new FileOutputStream(new File(filePath)));
            IOUtils.write(encoderStr.getBytes(charset), os);
        } catch (FileNotFoundException e) {
            throw new BusinessException(AppHttpStatus.READ_RESOURSE_NOT_FOUND_EXCEPTION.getStatus(), StringUtils.join("指定路径", filePath, "的文件不存在," + e));
        } catch (UnsupportedEncodingException e) {
            throw new BusinessException(AppHttpStatus.ENCODING_UNSUPPORTED_EXCEPTION);
        }
    }

    /**
     * URL编码，将指定文件的内容解码为字符串
     *
     * @param filePath 指定文件路径
     * @param charset  字符编码
     * @return 解码后的字符串
     */
    public static String urlDecoderWrap(String filePath, String charset) {
        try {
            if (StringUtils.isEmpty(charset)) {
                charset = CharsetUtils.UTF_8;
            }
            InputStream is = Base64.getUrlDecoder().wrap(new FileInputStream(new File(filePath)));
            return IOUtils.toString(is, charset);
        } catch (FileNotFoundException e) {
            throw new BusinessException(AppHttpStatus.READ_RESOURSE_NOT_FOUND_EXCEPTION.getStatus(), StringUtils.join("指定路径", filePath, "的文件不存在," + e));
        }
    }

    /**
     * MINE编码，将指定的字符串使用base64编码存入指定的文件
     *
     * @param filePath   指定文件路径
     * @param encoderStr 编码字符串
     * @param charset    字符编码
     */
    public static void mineEncoderWrap(String filePath, String encoderStr, String charset) {
        try {
            if (StringUtils.isEmpty(charset)) {
                charset = CharsetUtils.UTF_8;
            }
            OutputStream os = Base64.getMimeEncoder().wrap(new FileOutputStream(new File(filePath)));
            IOUtils.write(encoderStr.getBytes(charset), os);
        } catch (FileNotFoundException e) {
            throw new BusinessException(AppHttpStatus.READ_RESOURSE_NOT_FOUND_EXCEPTION.getStatus(), StringUtils.join("指定路径", filePath, "的文件不存在," + e));
        } catch (UnsupportedEncodingException e) {
            throw new BusinessException(AppHttpStatus.ENCODING_UNSUPPORTED_EXCEPTION);
        }
    }

    /**
     * MINE编码，将指定文件的内容解码为字符串
     *
     * @param filePath 指定文件路径
     * @param charset  字符编码
     * @return 解码后的字符串
     */
    public static String mineDecoderWrap(String filePath, String charset) {
        try {
            if (StringUtils.isEmpty(charset)) {
                charset = CharsetUtils.UTF_8;
            }
            InputStream is = Base64.getMimeDecoder().wrap(new FileInputStream(new File(filePath)));
            return IOUtils.toString(is, charset);
        } catch (FileNotFoundException e) {
            throw new BusinessException(AppHttpStatus.READ_RESOURSE_NOT_FOUND_EXCEPTION.getStatus(), StringUtils.join("指定路径", filePath, "的文件不存在," + e));
        }
    }
}

```

GitHub地址：[https://github.com/mingyang66/spring-parent/tree/master/doc/base](https://github.com/mingyang66/spring-parent/tree/master/doc/base)