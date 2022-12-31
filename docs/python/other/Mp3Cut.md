# Mp3Cut

---

```python
import sys
from mutagen.mp3 import MP3


"""
需: pip install mutagen
"""


class Mp3Cut(object):
    def __init__(self):
        # 读取歌曲名称
        self.file = sys.argv[1]

        # 获取比特率(kbps), kb是1000bit, p是per, s是second, 即每秒传输的数据量
        mp3 = MP3('王力宏 - 天地龙鳞.mp3')
        bitrate = mp3.info.bitrate

        # kb / 8 就是比特(bit)到字节(Byte)的转换, 即每秒传输多少字节
        self.kBps = bitrate / 8

        # 获取文件名（不包括文件扩展名）
        self.name = self.file[:self.file.rindex('.')]
    
    def start_time(self):
        tt = input("请输入开始时间, 分秒以.分隔: ")
        minute = int(tt.split('.')[0])  # 取分
        second = int(tt.split('.')[1])  # 取秒
        self.tms = minute * 60 + second  # 一共多少秒
        
    def stop_time(self):
        pt = input("请输入结束时间, 分秒以.分隔: ")
        minute = int(pt.split('.')[0])
        second = int(pt.split('.')[1])
        self.pms = minute * 60 + second
        
    def cut_mp3(self):
        first_part = self.tms * self.kBps  # 秒数(s) * 速率(kBps) = 数据大小(kB), 这就是开始截取的位置
        last_part = (self.pms - self.tms) * self.kBps  # 同理, 这就是结束截取的位置

        with open(self.file, 'rb') as start_f:
            start_f.read(int(first_part))  # 读取到要截取的位置
            with open(f'{self.name}_cut.mp3', 'wb') as stop_f:
                stop_f.write(start_f.read(int(last_part)))  # 在新文件写入到要结束截取的位置

def main():
    mp3 = Mp3Cut()
    mp3.start_time()
    mp3.stop_time()
    mp3.cut_mp3()
    
    
if __name__ == '__main__':        
    main()
```

运行:

```
python a.py xxxx.mp3
请输入开始时间（分秒以.分隔）：0.50
请输入结束时间（分秒以.分隔）：1.20
```

输出:

即可截取本首歌曲从0分50秒到1分20中之间的30秒长度的歌曲