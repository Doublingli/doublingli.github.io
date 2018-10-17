# Python中使用DES加密（PKCS5Padding）

今天在Python里面简单的做了一下DES算法需要的padding方法



Java方面使用的Padding方式是PKCS5，按照标准，PKCS5Pading的方式是在加密的字符串后面补齐，使得加密字符串的长度为8的整数倍

比如ABCDEF长度为6，需要补两个字符，如果采用0Padding的方式则缺多少就补多少'\x00'

现在需要的是PKCS5Padding，简单的说就是需要补几个就补几个几

8-6 = 2 需要补2个，就补2个2（'\x02')

于是加密前的字符串为'A' 'B' 'C' 'D' 'E' 'F' '\x02' '\x02'



特殊的是如果刚好是长度是8的整数倍的时候仍然需要再补8个8('\x08')，

如ABCDEFGH，需要补成 'A' 'B' 'C' 'D' 'E' 'F' 'G' 'H' '\x08' '\x08' '\x08' '\x08' '\x08' '\x08' '\x08' '\x08'

之前没想清楚为什么，直到做解密部分的时候才想清楚。

因为如果不这么做，万一加密前的字符串刚好就是以'\x01'结尾，或者'\x02' '\x02' 这样的字符结尾，并且长度刚好是8的整数倍的时候，解密的时候就出现二义性了。

如果加密前的字符串刚好以'x01',那么解密的时候就会当成这是一个padding，而将其去掉，



规定了如果长度是8的整数倍的时候需要再补8个'\x08'，则可以避免这样的问题。





```python
from binascii import b2a_hex, a2b_hex
from Crypto.Cipher import DES

DES_CRYPTO_UNIT_LENGTH = 8

DES_CRYPTO_KEY = "12345678"  #这个Key如果超过8个字符，多余的部分会被丢弃，不会影响结果


def encrypt_by_des(id_to_be_encrypted):
    '''
    	DES 加密
    	使用KPCS5进行padding
    '''
    # 设置加密密钥，初始化加密器
    e = DES.DESCipher(config.DES_CRYPTO_KEY)
    # 原字符串长度
    length = len(id_to_be_encrypted)
    # 计算padding长度
    padding_length = DES_CRYPTO_UNIT_LENGTH - length % DES_CRYPTO_UNIT_LENGTH
    # 根据padding长度 设置padding
    padding = chr(padding_length)
    # 补齐字符串
    id_with_padding = id_to_be_encrypted + (padding * padding_length)
    # 对padding后的字符串进行加密
    encrypted_byte_array = e.encrypt(id_with_padding)
    # 将加密后的byte数组转换成16进制字符串
    encrypted_hex_string = b2a_hex(encrypted_byte_array)
    return encrypted_hex_string



def decrypt_by_des(id_to_be_decrypted):
    '''
    	DES 解密
    	使用KPCS5进行padding
    '''
    # 设置加密密钥，初始化加密器
    e = DES.DESCipher(config.DES_CRYPTO_KEY)
    # 将十六进制密文转换成byte数组
    encrypted_byte_array = a2b_hex(id_to_be_decrypted)
    # 解密 获得带有padding的明文字符串
    decrypted_id_with_padding = e.decrypt(encrypted_byte_array)
    # 获取最后一位，也就是padding的每一个单元的字符
    padding = decrypted_id_with_padding[len(decrypted_id_with_padding)-1]
    # 根据padding计算padding长度
    padding_length = ord(padding)
    # 校验padding的合法性
    if padding_length > 8:
        LOGGER.error("PADDING LENGTH should not exceed 8")
        return None
    if decrypted_id_with_padding.endswith(padding*padding_length):
        LOGGER.error("PADDING MODE INVALID")
        return None
    # 移除padding得到明文
    decrypted_id = decrypted_id_with_padding[:len(decrypted_id_with_padding) - padding_length]
    return decrypted_id

```





