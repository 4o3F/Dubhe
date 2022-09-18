---
title: NPU第三届探索解密趣味密码比赛WriteUp
tags:
- CTF
- MISC
date: 2022-09-017 00:00:00
cover: /img/archives/ctfqwqcc.jpg
---

> 这是我第一次打CTF，所以也没啥成绩，各位权当一乐

#### 签到

按照提示得到flag{Welcome!}

#### 百家姓

百家姓解密`http://www.atoolbox.net/Tool.php?Id=1050`

#### 小猪别跑

培根密码解密

#### BASE233

根据源码中

`static const string base233_chars = "0B7UDomA2ZtJaOFdje43G9zRX1HTnfkhwqElc{WuLPI}bsypCY5SKrQN8gMiVvx6";`

`base233_chars`也是64个字符，同时向下看

```
            char_array_4[0] = (char_array_3[0] & 0xfc) >> 2;
            char_array_4[1] = ((char_array_3[0] & 0x03) << 4) + ((char_array_3[1] & 0xf0) >> 4);
            char_array_4[2] = ((char_array_3[1] & 0x0f) << 2) + ((char_array_3[2] & 0xc0) >> 6);
            char_array_4[3] = char_array_3[2] & 0x3f;
          
```

4个一组解密，推测是换表base64加密

利用 [http://web.chacuo.net/netbasex](http://web.chacuo.net/netbasex) 解密即可

#### 我是福瑞兽

兽音解密

#### PAPER

拼接图片

![paper-1](/img/archives/ctfqwqcc/paper-1.png)

摆放花枝

![paper-2](/img/archives/ctfqwqcc/paper-2.png)

根据题目所说，由下向上组成SIKMB，计算hash

#### 图书馆的水下

winhex打开

![underwater-1](/img/archives/ctfqwqcc/underwater-1.png)

更改0为9

![underwater-2](/img/archives/ctfqwqcc/underwater-2.png)

保存后再打开

![underwater-3](/img/archives/ctfqwqcc/underwater-3.png)

确实是水下哈

#### Notes

千千秀字解密

#### 隐约雷鸣

7zip直接解压得

#### 黑客小子

确定前5位后爆破起来容易很多

```pythoon
import hashlib

string = "86176"

for num in range(0,99999999):
    tempstr = str(num).zfill(8)
    if (hashlib.sha256((string+tempstr).encode('utf-8')).hexdigest()) == 'd64935ee63aac7d30a734395699093b954ab1487ea4e28fa16cf7c82d44513dc':
        print(tempstr)
        print(hashlib.sha256((string+tempstr).encode('utf-8')).hexdigest())
        break
    else:
        print(tempstr)


```

寄，就不能晚点给hint么，刚费半天劲查到176就给了个hint

#### 白鸟过河滩

观察txt，全为base64，逐行解密发现均为歌词，考虑base64隐写

```
base64chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/'

flag = ''
with open('in.txt', 'r') as f:
    for line in f.readlines():
        line = line[:-1]
        num = line.count('=')
        if num == 0:
            continue
        lastchar = line[-(num + 1)]

        myindex = base64chars.index(lastchar)
        bin_str = bin(myindex)[2:].zfill(6)
        flag += bin_str[6 - 2 * num:]
print(''.join([chr(int(flag[i:i + 8], 2)) for i in range(0, len(flag), 8)]))
```



#### easyrsa

根据p^q和p*q计算出p和q的值，求解代码如下

```
import math
import sys

def check_cong(k, p, q, n, xored=None):
    kmask = (1 << k) - 1
    p &= kmask
    q &= kmask
    n &= kmask
    pqm = (p*q) & kmask
    return pqm == n and (xored is None or (p^q) == (xored & kmask))

def extend(k, a):
    kbit = 1 << (k-1)
    assert a < kbit
    yield a
    yield a | kbit

def factor(n, p_xor_q):
    tracked = set([(p, q) for p in [0, 1] for q in [0, 1]
                   if check_cong(1, p, q, n, p_xor_q)])

    PRIME_BITS = int(math.ceil(math.log(n, 2)/2))

    maxtracked = len(tracked)
    for k in range(2, PRIME_BITS+1):
        newset = set()
        for tp, tq in tracked:
            for newp_ in extend(k, tp):
                for newq_ in extend(k, tq):
                    newp, newq = sorted([newp_, newq_])
                    if check_cong(k, newp, newq, n, p_xor_q):
                        newset.add((newp, newq))

        tracked = newset
        if len(tracked) > maxtracked:
            maxtracked = len(tracked)
    print('Tracked set size: {} (max={})'.format(len(tracked), maxtracked))

    for p, q in tracked:
        if p != 1 and p*q == n:
            return p, q

    assert False, 'factors were not in tracked set. Is your p^q correct?'

def main():
    if len(sys.argv) != 3:
        print('Usage: xor_factor.py n p_xor_q', file=sys.stderr)
        print('(give both numbers in decimal)', file=sys.stderr)

    n = int(sys.argv[1])
    p_xor_q = int(sys.argv[2])

    p, q = factor(n, p_xor_q)
    print(p)
    print(q)

if __name__ == '__main__':
    main()
```



解密代码如下

```python
from Crypto.Util.number import getPrime, bytes_to_long, long_to_bytes


def eucalg(a, b):
 swapped = False
 if a < b:
  a, b = b, a
  swapped = True
 ca = (1, 0)
 cb = (0, 1)
 while b != 0:
  k = a // b
  a, b, ca, cb = b, a-b*k, cb, (ca[0]-k*cb[0], ca[1]-k*cb[1])
 if swapped:
  return (ca[1], ca[0])
 else:
  return ca

def modpow(b, e, n):
 tst = 1
 siz = 0
 while e >= tst:
  tst <<= 1
  siz += 1
 siz -= 1
 r = 1
 for i in range(siz, -1, -1):
  r = (r * r) % n
  if (e >> i) & 1: r = (r * b) % n
 return r

p = getPrime(1024)
q = getPrime(1024)

p = 92597391608572719046497582254246412516971017760101273266665552882430391051182993916925443981602163939260401276717839577932884562125259391023408513455594637080458988334026610444045354576275440673040859586278093584638158760487611282643862973538059918500825486614917202195301729524228623574607585184167942181761
q = 118292804276217349447191099265687316587905393506161146013033646686812953382380863621066608010669164684775802453856883877341366260752536413982418249492465161734274854442013968867696857720236520896454772468189907263687416557608012275275293816050740695775710842394100992549535275907921216237115265042326380248121

encrypted = int(0x38123bfd6acb232cf735c837f1dca86d6a0a8bc58b03db61526287b649dd5351acd99df020f09977ca460d433d934f3c2ad612d7637578a599a6bb1cebbcda900e93b1778ec5a26c35f09b0e133760fd5020e7d11134b544b232302c8bc5fde49d16147fb4bc856f355b3115dbfa841b0c2decf9beb294268a4ffb72ec90f0867bfaac48e901458d8fed0fc426ee15de79228c8d0df016e64484d8f146fd1fcf1dd49cab0523940d8ef737f78b9871fd2ef5da28da20bc2f70e3ab352173f895e80435ceb6297076ddccdb85b35546c3dd787be1812a8a52dff5f59a96726c00d6cbdc462a59b8e95d6106daf523c1e08f0518d08945c31f0700866e8b08467d)

n = p * q
e = 65537
x = p ^ q

lambda_n = (p - 1) * (q - 1)

d = eucalg(e, lambda_n)[0]
if d < 0: d += lambda_n

result = modpow(encrypted, d, n)

print(result)
print(long_to_bytes(result).decode("utf-8"))

```

#### 键盘中的爱讯号

根据摩斯密码转换，根据换行可知分成5组，然后改过题后可以在键盘上清楚地看出来形状

#### 隐藏在主播背后的秘密

不想像我一样走弯路可以直接零宽度字符隐写

像我一样的话，就对js进行解密，按照网上大佬的方法拿python日jsjiami.com.v5得到

````
 var _0x1d1742 = null;
 document['addEventListener']('DOMContentLoaded', function() {
     _0x1d1742 = new Date()['getTime']();
 });
 document['getElementById']('check')['addEventListener']('click', function() {
     var _0x5d9d1c = {
         'WBqsH': function _0x566381(_0x1edce4, _0x4a7e5d) {
             return _0x1edce4 > _0x4a7e5d;
         },
         'tNJlv': function _0x594a3b(_0x134cb2, _0x3d9665) {
             return _0x134cb2 - _0x3d9665;
         },
         'VOjTh': function _0x3b6d8f(_0xab4de8, _0x1b7238) {
             return _0xab4de8 * _0x1b7238;
         },
         'QMmCJ': function _0x17f150(_0x10d945, _0x55f605) {
             return _0x10d945 * _0x55f605;
         },
         'SMbkp': 'flagDiv',
         'iZhDq': 'block',
         'dJvSW': 'flag',
         'cAzdn': 'B﻿‎‌​﻿‏‍​‍﻿‌​﻿‎‍​﻿﻿‍​‌﻿‏​‏﻿‍​‏﻿‍​﻿‎‎​‍﻿‎​‍‎‍​‏﻿﻿​﻿‎‍​﻿‎﻿​﻿‍‏​‍﻿‎​‍‎‌​﻿‏‎​‍﻿﻿​‍﻿‌​‏‎‎‎站关注"快乐滋崩在这里"啊喵( •̀ ω •́ )✧！',
         'BJYLW': function _0x5cc7e5(_0x8d3f9f, _0xbc9494) {
             return _0x8d3f9f(_0xbc9494);
         },
         'FkQCo': '你还没听完呢，休想骗我╰（‵□′）╯'
     };
     var _0x58a403 = new Date()['getTime']();
     if (_0x5d9d1c['WBqsH'](_0x5d9d1c['tNJlv'](_0x58a403, _0x1d1742), _0x5d9d1c['VOjTh'](_0x5d9d1c['QMmCJ'](0x3e8, 0x3c), 0x3))) {
         document['getElementById'](_0x5d9d1c['SMbkp'])['style']['display'] = _0x5d9d1c['iZhDq'];
         document['getElementById'](_0x5d9d1c['dJvSW'])['innerText'] = _0x5d9d1c['cAzdn'];
     } else {
         _0x5d9d1c['BJYLW'](alert, _0x5d9d1c['FkQCo']);
     }
 });;
 (function(_0x294487, _0x16e04d, _0x1858c1) {
     var _0x50cae8 = {
         'OwEhz': '7|0|6|1|4|3|2|5',
         'gfTgK': function _0x5540e2(_0x3e8997) {
             return _0x3e8997();
         },
         'gzrAa': function _0x1cbc53(_0x4f1ca7, _0x58b6a2, _0x17cf75) {
             return _0x4f1ca7(_0x58b6a2, _0x17cf75);
         },
         'mpHaD': 'ert',
         'MlYXI': function _0x5c304d(_0x1534c5, _0x567774) {
             return _0x1534c5 !== _0x567774;
         },
         'vNsEX': 'undefined',
         'AokDu': function _0x506aa0(_0x5a7a48, _0x429507) {
             return _0x5a7a48 === _0x429507;
         },
         'Tbors': 'jsjiami.com.v5',
         'qIsyT': function _0x22246d(_0x65881a, _0x58b51d) {
             return _0x65881a + _0x58b51d;
         },
         'nghjg': '版本号，js会定期弹窗，还请支持我们的工作',
         'tgnZF': '删除版本号，js会定期弹窗',
         'Mzdck': function _0x20c3e0(_0x463d9d, _0x28b8f3, _0x38fc84) {
             return _0x463d9d(_0x28b8f3, _0x38fc84);
         }
     };
     var _0x1b08c2 = _0x50cae8['OwEhz']['split']('|'),
         _0x276135 = 0x0;
     while (!![]) {
         switch (_0x1b08c2[_0x276135++]) {
             case '0':
                 var _0x53844a = function() {
                     var _0x6116a3 = {
                         'bDoLZ': function _0x38538b(_0x572094, _0x29f149) {
                             return _0x572094 === _0x29f149;
                         },
                         'dSIKy': 'iZs',
                         'ilUbi': 'Eke',
                         'awuaY': 'flagDiv',
                         'uzqrX': 'block',
                         'GrFVE': 'flag',
                         'Xjrbq': 'B﻿‎‌​﻿‏‍​‍﻿‌​﻿‎‍​﻿﻿‍​‌﻿‏​‏﻿‍​‏﻿‍​﻿‎‎​‍﻿‎​‍‎‍​‏﻿﻿​﻿‎‍​﻿‎﻿​﻿‍‏​‍﻿‎​‍‎‌​﻿‏‎​‍﻿﻿​‍﻿‌​‏‎‎‎站关注"快乐滋崩在这里"啊喵( •̀ ω •́ )✧！'
                     };
                     if (_0x6116a3['bDoLZ'](_0x6116a3['dSIKy'], _0x6116a3['ilUbi'])) {
                         document['getElementById'](_0x6116a3['awuaY'])['style']['display'] = _0x6116a3['uzqrX'];
                         document['getElementById'](_0x6116a3['GrFVE'])['innerText'] = _0x6116a3['Xjrbq'];
                     } else {
                         var _0x486aaa = !![];
                         return function(_0x1c55a5, _0x3ebcb9) {
                             var _0x593be6 = {
                                 'tImxd': function _0x22eb7a(_0x50571d, _0x3808b9) {
                                     return _0x50571d !== _0x3808b9;
                                 },
                                 'wCxVj': 'YRt',
                                 'BcOhO': 'uZZ',
                                 'JbNhB': 'ert',
                                 'OxjYN': 'undefined',
                                 'kWULK': function _0x5d27ec(_0x4eada5, _0x53a038) {
                                     return _0x4eada5 === _0x53a038;
                                 },
                                 'ZZAGS': 'jsjiami.com.v5',
                                 'IJpzf': function _0x4d8aee(_0x3c7f6a, _0x101729) {
                                     return _0x3c7f6a + _0x101729;
                                 },
                                 'ECbpv': '版本号，js会定期弹窗，还请支持我们的工作',
                                 'vYpKP': 'GAm'
                             };
                             var _0x41db08 = _0x486aaa ? function() {
                                 if (_0x3ebcb9) {
                                     if (_0x593be6['tImxd'](_0x593be6['wCxVj'], _0x593be6['BcOhO'])) {
                                         var _0x59711c = _0x3ebcb9['apply'](_0x1c55a5, arguments);
                                         _0x3ebcb9 = null;
                                         return _0x59711c;
                                     } else {
                                         _0x1858c1 += _0x593be6['JbNhB'];
                                         _0x16e04d = encode_version;
                                         if (!(_0x593be6['tImxd'](typeof _0x16e04d, _0x593be6['OxjYN']) && _0x593be6['kWULK'](_0x16e04d, _0x593be6['ZZAGS']))) {
                                             _0x294487[_0x1858c1](_0x593be6['IJpzf']('删除', _0x593be6['ECbpv']));
                                         }
                                     }
                                 }
                             } : function() {
                                 if (_0x593be6['kWULK'](_0x593be6['vYpKP'], _0x593be6['vYpKP'])) {} else {
                                     debugger;
                                 }
                             };
                             _0x486aaa = ![];
                             return _0x41db08;
                         };
                     }
                 }();
                 continue;
             case '1':
                 var _0x4d2ddf = function() {
                     var _0x3ff9ac = {
                         'PjzUn': function _0x26d73b(_0xcf26b7, _0x33e84f) {
                             return _0xcf26b7 === _0x33e84f;
                         },
                         'rPyGz': 'mWs'
                     };
                     if (_0x3ff9ac['PjzUn'](_0x3ff9ac['rPyGz'], _0x3ff9ac['rPyGz'])) {
                         var _0x45a47e = !![];
                         return function(_0xca38a6, _0x4a8ce4) {
                             var _0x381eae = {
                                 'rqlpU': function _0x2f3056(_0x4c899b, _0x2a28de) {
                                     return _0x4c899b !== _0x2a28de;
                                 },
                                 'OrCJf': 'vvc',
                                 'SaJyR': function _0x3d8165(_0x58c55b, _0x59e98a) {
                                     return _0x58c55b !== _0x59e98a;
                                 },
                                 'oehLs': 'RYe'
                             };
                             if (_0x381eae['SaJyR'](_0x381eae['oehLs'], _0x381eae['oehLs'])) {
                                 var _0x2214bf = _0x45a47e ? function() {
                                     if (_0x4a8ce4) {
                                         var _0x527fa1 = _0x4a8ce4['apply'](_0xca38a6, arguments);
                                         _0x4a8ce4 = null;
                                         return _0x527fa1;
                                     }
                                 } : function() {};
                                 _0x45a47e = ![];
                                 return _0x2214bf;
                             } else {
                                 var _0x53e27f = _0x45a47e ? function() {
                                     if (_0x4a8ce4) {
                                         if (_0x381eae['rqlpU'](_0x381eae['OrCJf'], _0x381eae['OrCJf'])) {
                                             return debuggerProtection;
                                         } else {
                                             var _0x486f89 = _0x4a8ce4['apply'](_0xca38a6, arguments);
                                             _0x4a8ce4 = null;
                                             return _0x486f89;
                                         }
                                     }
                                 } : function() {};
                                 _0x45a47e = ![];
                                 return _0x53e27f;
                             }
                         };
                     } else {}
                 }();
                 continue;
             case '2':
                 _0x1858c1 = 'al';
                 continue;
             case '3':
                 _0x50cae8['gfTgK'](_0x878972);
                 continue;
             case '4':
                 var _0x878972 = _0x50cae8['gzrAa'](_0x4d2ddf, this, function() {
                     var _0x5a4746 = {
                         'BKwoc': function _0xb37f11(_0x9de7e5, _0x54a850) {
                             return _0x9de7e5 === _0x54a850;
                         },
                         'tzlAH': 'LRv',
                         'lgnDf': 'FPa',
                         'tfyGh': '删除版本号，js会定期弹窗',
                         'TEWuD': function _0x435134(_0x1ab030, _0x4d9c5b) {
                             return _0x1ab030 !== _0x4d9c5b;
                         },
                         'HbTNJ': 'undefined',
                         'rCRTM': 'object',
                         'TnQfv': 'function',
                         'yxFqI': '5|2|6|3|0|4|1'
                     };
                     if (_0x5a4746['BKwoc'](_0x5a4746['tzlAH'], _0x5a4746['lgnDf'])) {
                         _0x294487[_0x1858c1](_0x5a4746['tfyGh']);
                     } else {
                         var _0x116010 = function() {
                             var _0x1ce1e6 = {
                                 'Mvzmi': function _0x5d0f28(_0x2f2b14, _0xdb1e22) {
                                     return _0x2f2b14 !== _0xdb1e22;
                                 },
                                 'RcgUE': 'rhk',
                                 'OBQlo': 'Mhm',
                                 'sJXfk': function _0x429bf7(_0x102dfa, _0x536a2f) {
                                     return _0x102dfa(_0x536a2f);
                                 },
                                 'yyYkR': '你还没听完呢，休想骗我╰（‵□′）╯'
                             };
                             if (_0x1ce1e6['Mvzmi'](_0x1ce1e6['RcgUE'], _0x1ce1e6['OBQlo'])) {} else {
                                 _0x1ce1e6['sJXfk'](alert, _0x1ce1e6['yyYkR']);
                             }
                         };
                         var _0x1285db = _0x5a4746['TEWuD'](typeof window, _0x5a4746['HbTNJ']) ? window : _0x5a4746['BKwoc'](typeof process, _0x5a4746['rCRTM']) && _0x5a4746['BKwoc'](typeof require, _0x5a4746['TnQfv']) && _0x5a4746['BKwoc'](typeof global, _0x5a4746['rCRTM']) ? global : this;
                         if (!_0x1285db['console']) {
                             _0x1285db['console'] = function(_0x3a16f0) {
                                 var _0x15ee9e = {
                                     'MOiIM': function _0x323503(_0x3da515, _0x109788) {
                                         return _0x3da515 === _0x109788;
                                     },
                                     'ULbaz': 'eKy',
                                     'NhpAi': '2|8|4|0|6|3|7|1|5',
                                     'yIUdS': function _0x483510(_0x4bbda0) {
                                         return _0x4bbda0();
                                     }
                                 };
                                 if (_0x15ee9e['MOiIM'](_0x15ee9e['ULbaz'], _0x15ee9e['ULbaz'])) {
                                     var _0x31d6ef = _0x15ee9e['NhpAi']['split']('|'),
                                         _0x28b0fb = 0x0;
                                     while (!![]) {
                                         switch (_0x31d6ef[_0x28b0fb++]) {
                                             case '0':
                                                 _0x1858c1['debug'] = _0x3a16f0;
                                                 continue;
                                             case '1':
                                                 _0x1858c1['trace'] = _0x3a16f0;
                                                 continue;
                                             case '2':
                                                 var _0x1858c1 = {};
                                                 continue;
                                             case '3':
                                                 _0x1858c1['error'] = _0x3a16f0;
                                                 continue;
                                             case '4':
                                                 _0x1858c1['warn'] = _0x3a16f0;
                                                 continue;
                                             case '5':
                                                 return _0x1858c1;
                                             case '6':
                                                 _0x1858c1['info'] = _0x3a16f0;
                                                 continue;
                                             case '7':
                                                 _0x1858c1['exception'] = _0x3a16f0;
                                                 continue;
                                             case '8':
                                                 _0x1858c1['log'] = _0x3a16f0;
                                                 continue;
                                         }
                                         break;
                                     }
                                 } else {
                                     _0x15ee9e['yIUdS'](_0x32427f);
                                 }
                             }(_0x116010);
                         } else {
                             var _0x3586aa = _0x5a4746['yxFqI']['split']('|'),
                                 _0x474881 = 0x0;
                             while (!![]) {
                                 switch (_0x3586aa[_0x474881++]) {
                                     case '0':
                                         _0x1285db['console']['error'] = _0x116010;
                                         continue;
                                     case '1':
                                         _0x1285db['console']['trace'] = _0x116010;
                                         continue;
                                     case '2':
                                         _0x1285db['console']['warn'] = _0x116010;
                                         continue;
                                     case '3':
                                         _0x1285db['console']['info'] = _0x116010;
                                         continue;
                                     case '4':
                                         _0x1285db['console']['exception'] = _0x116010;
                                         continue;
                                     case '5':
                                         _0x1285db['console']['log'] = _0x116010;
                                         continue;
                                     case '6':
                                         _0x1285db['console']['debug'] = _0x116010;
                                         continue;
                                 }
                                 break;
                             }
                         }
                     }
                 });
                 continue;
             case '5':
                 try {
                     _0x1858c1 += _0x50cae8['mpHaD'];
                     _0x16e04d = encode_version;
                     if (!(_0x50cae8['MlYXI'](typeof _0x16e04d, _0x50cae8['vNsEX']) && _0x50cae8['AokDu'](_0x16e04d, _0x50cae8['Tbors']))) {
                         _0x294487[_0x1858c1](_0x50cae8['qIsyT']('删除', _0x50cae8['nghjg']));
                     }
                 } catch (_0x392609) {
                     _0x294487[_0x1858c1](_0x50cae8['tgnZF']);
                 }
                 continue;
             case '6':
                 (function() {
                     _0x3e527c['GTiEt'](_0x53844a, this, function() {
                         var _0x3096a3 = {
                             'bigJW': function _0xf1f726(_0x1c0a65, _0x3e2a23) {
                                 return _0x1c0a65 === _0x3e2a23;
                             },
                             'ihTYb': 'mwZ',
                             'Gdvio': 'function *\( *\)',
                             'xUbcH': '\+\+ *(?:_0x(?:[a-f0-9]){4,6}|(?:\b|\d)[a-z0-9]{1,4}(?:\b|\d))',
                             'PivQW': function _0x4d0a28(_0x5b9777, _0x349b38) {
                                 return _0x5b9777(_0x349b38);
                             },
                             'GUhoH': 'init',
                             'vAhFQ': function _0x140234(_0x1335a3, _0x29162f) {
                                 return _0x1335a3 + _0x29162f;
                             },
                             'hgHXM': 'chain',
                             'kxbKG': 'input',
                             'cNncB': function _0x1388a0(_0x816c7c, _0x2ed125) {
                                 return _0x816c7c(_0x2ed125);
                             },
                             'kNdkS': function _0x4bf9c4(_0x3cf2de) {
                                 return _0x3cf2de();
                             }
                         };
                         if (_0x3096a3['bigJW'](_0x3096a3['ihTYb'], _0x3096a3['ihTYb'])) {
                             var _0x7f766a = new RegExp(_0x3096a3['Gdvio']);
                             var _0x1e1557 = new RegExp(_0x3096a3['xUbcH'], 'i');
                             var _0xa38888 = _0x3096a3['PivQW'](_0x32427f, _0x3096a3['GUhoH']);
                             if (!_0x7f766a['test'](_0x3096a3['vAhFQ'](_0xa38888, _0x3096a3['hgHXM'])) || !_0x1e1557['test'](_0x3096a3['vAhFQ'](_0xa38888, _0x3096a3['kxbKG']))) {
                                 _0x3096a3['cNncB'](_0xa38888, '0');
                             } else {
                                 _0x3096a3['kNdkS'](_0x32427f);
                             }
                         } else {
                             _0x3096a3['cNncB'](debuggerProtection, 0x0);
                         }
                     })();
                 }());
                 continue;
             case '7':
                 var _0x3e527c = {
                     'GTiEt': function _0x5e49ba(_0x1ab64a, _0xb52383, _0x2e695f) {
                         return _0x50cae8['Mzdck'](_0x1ab64a, _0xb52383, _0x2e695f);
                     }
                 };
                 continue;
         }
         break;
     }
 }(window));
 
 function _0x32427f(_0x36e44e) {
     var _0xb301cd = {
         'vrvBf': function _0x18aefa(_0x56887f, _0x2d34ee) {
             return _0x56887f === _0x2d34ee;
         },
         'LNysv': 'string',
         'Kcypy': function _0x49d3e2(_0x7be14e, _0x2bf6ab) {
             return _0x7be14e !== _0x2bf6ab;
         },
         'OkIjm': 'UkB',
         'yBwFo': 'rYm',
         'vSHew': function _0xada4e(_0x3529df) {
             return _0x3529df();
         },
         'YGfuq': function _0xa3fd09(_0xe28da8, _0x4e6f2f) {
             return _0xe28da8 + _0x4e6f2f;
         },
         'hGsYZ': function _0x314a68(_0x4a669e, _0x56714f) {
             return _0x4a669e / _0x56714f;
         },
         'dUdsF': 'length',
         'RiqMF': function _0x280fd3(_0x24c20f, _0x103400) {
             return _0x24c20f === _0x103400;
         },
         'nlgTD': function _0x54eab0(_0x1a4d82, _0x2a83ab) {
             return _0x1a4d82 % _0x2a83ab;
         },
         'sLkfm': function _0x524150(_0x2de848, _0x3c5ce9) {
             return _0x2de848 !== _0x3c5ce9;
         },
         'gEddV': 'ulR',
         'krdBF': 'Ake',
         'AemHp': function _0x468bdc(_0x179de3, _0x5e0586) {
             return _0x179de3(_0x5e0586);
         },
         'FfmAe': 'zks',
         'QWbpF': 'CAU'
     };
 
     function _0x1fde12(_0x2034ba) {
         if (_0xb301cd['vrvBf'](typeof _0x2034ba, _0xb301cd['LNysv'])) {
             if (_0xb301cd['Kcypy'](_0xb301cd['OkIjm'], _0xb301cd['yBwFo'])) {
                 var _0x14527e = function() {
                     var _0x44c689 = {
                         'MJSKX': function _0xa62e86(_0x3a52d0, _0x3cb759) {
                             return _0x3a52d0 === _0x3cb759;
                         },
                         'PVZXp': 'Ktm',
                         'rqqDv': function _0x348a1d(_0x44cbf3, _0x1b41bf) {
                             return _0x44cbf3 + _0x1b41bf;
                         },
                         'FIowJ': '版本号，js会定期弹窗，还请支持我们的工作'
                     };
                     while (!![]) {
                         if (_0x44c689['MJSKX'](_0x44c689['PVZXp'], _0x44c689['PVZXp'])) {} else {
                             w[c](_0x44c689['rqqDv']('删除', _0x44c689['FIowJ']));
                         }
                     }
                 };
                 return _0xb301cd['vSHew'](_0x14527e);
             } else {}
         } else {
             if (_0xb301cd['Kcypy'](_0xb301cd['YGfuq']('', _0xb301cd['hGsYZ'](_0x2034ba, _0x2034ba))[_0xb301cd['dUdsF']], 0x1) || _0xb301cd['RiqMF'](_0xb301cd['nlgTD'](_0x2034ba, 0x14), 0x0)) {
                 debugger;
             } else {
                 if (_0xb301cd['sLkfm'](_0xb301cd['gEddV'], _0xb301cd['krdBF'])) {
                     debugger;
                 } else {}
             }
         }
         _0xb301cd['AemHp'](_0x1fde12, ++_0x2034ba);
     }
     try {
         if (_0x36e44e) {
             return _0x1fde12;
         } else {
             _0xb301cd['AemHp'](_0x1fde12, 0x0);
         }
     } catch (_0x32790f) {
         if (_0xb301cd['sLkfm'](_0xb301cd['FfmAe'], _0xb301cd['QWbpF'])) {} else {
             _0x1d1742 = new Date()['getTime']();
         }
     }
 }
 setInterval(function() {
     var _0x3b7ad1 = {
         'mLFez': function _0x5c86e1(_0x407f27) {
             return _0x407f27();
         }
     };
     _0x3b7ad1['mLFez'](_0x32427f);
 }, 0xfa0);;
 encode_version = 'jsjiami.com.v5';
````

在console中给`_0x1d1742`赋值为0跳过3分钟......

亏得我还分析了好久下面这一堆b代码干啥的，原来就是个自我校验.........

然后发现B后面一长串unicode，结合歌曲名林宽，解出
