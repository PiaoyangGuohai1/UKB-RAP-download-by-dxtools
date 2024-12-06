# UKB-RAP-download-by-dxtools
How to download UKB data by dxtools

直接从 [UKB RAP](https://ukbiobank.dnanexus.com/login) 上下载临床数据时，需要手动点击 filed 进行下载，比较麻烦

UKB RAP平台提供了dxtools 工具有助于我们进行命令行式的交互，完成数据的快速下载与分析。

这里提供了 DNAnexus 官方的操作视频：[DNAnexus - YouTube](https://www.youtube.com/@Dnanexus)

## 1 安装dxtools

说明文档：[Downloads | DNAnexus Documentation](https://documentation.dnanexus.com/downloads)

```shell
pip3 install dxpy
```

安装完之后可以进行登录和退出等操作，输入预先准备好的账户密码。

```shell
dx login # 登录
dx -h # 查看帮助文档
dx logout # 登出
```

## 2 直接下载RAP平台的数据，譬如已经使用table-explore完成了提取后的txt文件

前往RAP平台赋值项目名-路径。

```shell
dx download 'project-GJpxxxxxxxxxxxxxx3Fkq4Y:/lxy/field/1013_C1712_2414.txt'
dx upload filename
```

## 3 提供field ID完成表型数据的提取+下载

### 3.1 首先需要准备一些信息

#### cohort_dataset信息

需要先前往UKB RAP平台获得，由 project id 和 record id 组成，具体操作方法：前往 RAP 平台找到想要下载的人群队列，然后复制 path 得到 project id；复制 ID 得到 record id。

#### **Fields 文件**：包含我们想要提取的 fields

当查询的 fileds 比较多时，从推荐采用 `--fields-file` 参数，该参数需要输入一个文档，每一行为一个 field，每个 field 前需要指定该 field 的 entidy。

查询全部可用的 entidy

```shell
dx extract_dataset --list-entities app88xxx_202211xxxxxxxxx.dataset
# covid19_result_england  COVID19 Test Result Record (England)
# covid19_result_scotland COVID19 Test Result Record (Scotland)
# covid19_result_wales    COVID19 Test Result Record (Wales)
# death   Death Record
# death_cause     Death Cause Record
# gp_clinical     GP Clinical Event Record
# gp_registrations        GP Registration Record
# gp_scripts      GP Prescription Record
# hesin   Hospitalization Record
# hesin_critical  Hospital Critical Care Record
# hesin_delivery  Hospital Delivery Record
# hesin_diag      Hospital Diagnosis Record
# hesin_maternity Hospital Maternity Record
# hesin_oper      Hospital Operation Record
# hesin_psych     Hospital Psychiatric Detention Record
# participant     Participant
```

* 查询全部可用的 fields

```shell
dx extract_dataset --list-fields app88xxx_202211xxxxxxxxx.dataset > all_fields.txt
```

从上面得到的 all_fields 文件中提取想要的 fields，就可以构建 fields_file 文件了

每一个 fields_file 的内容如下所示：

![image.png](https://image-1300560293.cos.ap-guangzhou.myqcloud.com/markdown/202410151438403.png)

在 windows 上**运行的 powershell 代码，linux 上需要修改

1. 检查 ".\提取的 fields" 下有哪些需要提取的 fields
2. 如果 result 文件夹下没有对应的 csv 文件，则进行下

> 注意：每个 fields 文件最好不要太多（超过 60 个），否则会下载失败  
> 注意：每个 fields 文件后面不要留空行！

### 3.2 执行提取

* 如果不加 `-ddd` 参数，则不会生成多个说明文件，只生成结果文件

```shell
dx extract_dataset project-GJpxxxxxxxxxxxFkq4Y:record-GJqxxxxxxxxxxxxxz2 --fields-file ".\fields\提取的fields\C1712_p131270-p131319.txt" --delimiter "," --output ".\fields\result\C1712_p131270-p131319.csv"
```

* 如果添加 `-ddd` 参数，需要修改 output 参数为文件夹的名字，而不是文件名。此时，会在该文件夹下生成该 database 的 coding、data_dictionary 和 entity_dictionary 文件。

```shell
dx extract_dataset project-GJpxxxxxxxxxxxFkq4Y:record-GJqxxxxxxxxxxxxxz2 -ddd --fields-file ".\fields\提取的fields\C1712_p131270-p131319.txt" --delimiter "," --output "./output_directory"
```



### 3.3 批量执行

```powershell
# 设置文件夹路径
cd D:\OBSIDIAN\03_课题进度\2022_PA\BPandCKD\UKBiobank\fields
$fieldsFolder = ".\提取的fields" # 这里存放要提取的fields文件
$resultFolder = ".\result" # 这里存放结果

# 遍历 提取的fields 文件夹中的所有 .txt 文件
foreach ($file in Get-ChildItem "$fieldsFolder\*.txt") {
    
    # 获取当前文件的文件名（不带扩展名），例如 C1712_2401_1
    $filenameWithoutExtension = [System.IO.Path]::GetFileNameWithoutExtension($file.FullName)
    
    # 检查是否有对应的 .csv 文件存在
    $csvFile = "$resultFolder\$filenameWithoutExtension.csv"
    
    if (-not (Test-Path $csvFile)) {
        # 如果没有对应的 .csv 文件，则执行 dx extract_dataset 命令
        Write-Host "Extracting dataset for $filenameWithoutExtension..."

        dx extract_dataset project-GJpxxxxxxxxxxxFkq4Y:record-GJqxxxxxxxxxxxxxz2 `
            --fields-file "$($file.FullName)" `
            --delimiter "," `
            --output "$csvFile"
        
        Write-Host "Dataset extracted to $csvFile"
    } else {
        Write-Host "CSV file $csvFile already exists. Skipping."
    }
}
```


有时候会出现网络问题导致失败，重新运行即可。







