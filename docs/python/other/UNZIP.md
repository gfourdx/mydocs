# UNZIP

---

```python
import pathlib
import sys
import pathlib
import zipfile
 

def un_zip_file():
    """
    解压多个ZIP文件
    python un_zip.py file1.zip file2.zip file3.zip ...
    解压后生成文件夹 file1 file2 file3 ...
    每个文件夹中就是zip解压后的文件
    """
    zip_files = sys.argv[1:]

    for zip_file in zip_files:
        # 获取 去除掉 .zip 后的名称
        dir_nbame = zip_file[:-4]
        # 用上面的名称当作文件夹的名称
        if not pathlib.Path(dir_nbame).is_dir():
            pathlib.Path(dir_nbame).mkdir()
        
        with zipfile.ZipFile(zip_file, "r") as f:
            file_list = f.namelist()
            print(f'压缩文件 {zip_file} 一共有 {len(file_list)} 个: ')
            for index, file in enumerate(file_list):
                print(f'{index+1}: {file}')

            un_zip_file_number = int(input('请输入要解压的文件(输入0解压所有): '))
            if un_zip_file_number > len(file_list):
                print('编号错误')
                continue
            if un_zip_file_number == 0:
                f.extractall(path=dir_nbame)
            else:
                f.extract(file_list[un_zip_file_number-1], path=dir_nbame)


def main():
    un_zip_file()


if __name__ == '__main__':
    main()
```

