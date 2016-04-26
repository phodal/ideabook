Python制作照片地图
===

最后效果可参见

[Phodal|cart db][1]

##EXIF##

> 可交换图像文件常被简称为EXIF（Exchangeable image file format），是专门为数码相机的照片设定的，可以记录数码照片的属性信息和拍摄数据。

EXIF信息以0xFFE1作为开头标记，后两个字节表示EXIF信息的长度。所以EXIF信息最大为64 kB，而内部采用TIFF格式。

###ExifRead###

来自官方的简述

> **Python library to extract EXIF data from tiff and jpeg files.**

###ExifRead安装###

    pip install exifread

###ExifRead Exif.py###

官方写了一个exif.py的command可直接查看照片信息

     EXIF.py images.jpg


##CartoDB##

###简介 ###

Create dynamic maps, analyze and build location aware and geospatial applications with your data using the power using the power of PostGIS in the cloud.

简单的来说，就是我们可以创建包含位置信息的内容到上面去。

![Phodal's Image][2]

  [1]: http://phodal.cartodb.com/viz/80484668-b165-11e3-be2e-0e73339ffa50/public_map
  [2]: /static/media/uploads/screen_shot_2014-03-22_at_10.05.39_am.jpg

##打造自己的照片地图##
主要步骤如下

 - 需要遍历自己的全部图片文件，
 - 解析照片信息
 - 生成地理信息文件
 -  上传到cartodb

###python 遍历文件###
代码如下，来自于《python cookbook》

    import os, fnmatch
    def all_files(root, patterns='*', single_level=False, yield_folders=False):
        patterns = patterns.split(';')
        for path, subdirs, files in os.walk(root):
            if yield_folders:
                files.extend(subdirs)
            files.sort()
            for name in files:
                for pattern in patterns:
                    if fnmatch.fnmatch(name, pattern):
                        yield os.path.join(path, name)
                        break
                    if single_level:
                        break

###python 解析照片信息###
由于直接从照片中提取的信息是

    [34, 12, 51513/1000]

也就是

    N 34� 13' 12.718

几度几分几秒的形式，我们需要转换为

    34.2143091667

具体的大致就是

    def parse_gps(titude):
        first_number = titude.split(',')[0]
        second_number = titude.split(',')[1]
        third_number = titude.split(',')[2]
        third_number_parent = third_number.split('/')[0]
        third_number_child = third_number.split('/')[1]
        third_number_result = float(third_number_parent) / float(third_number_child)
        return float(first_number) + float(second_number)/60 + third_number_result/3600

也就是我们需要将second/60，还有minutes/3600。

###python 提取照片信息生成文件###

    import json
    import exifread
    import os, fnmatch
    from exifread.tags import DEFAULT_STOP_TAG, FIELD_TYPES
    from exifread import process_file, __version__

    def all_files(root, patterns='*', single_level=False, yield_folders=False):
        patterns = patterns.split(';')
        for path, subdirs, files in os.walk(root):
            if yield_folders:
                files.extend(subdirs)
            files.sort()
            for name in files:
                for pattern in patterns:
                    if fnmatch.fnmatch(name, pattern):
                        yield os.path.join(path, name)
                        break
                    if single_level:
                        break

    def parse_gps(titude):
        first_number = titude.split(',')[0]
        second_number = titude.split(',')[1]
        third_number = titude.split(',')[2]
        third_number_parent = third_number.split('/')[0]
        third_number_child = third_number.split('/')[1]
        third_number_result = float(third_number_parent) / float(third_number_child)
        return float(first_number) + float(second_number)/60 + third_number_result/3600

    jsonFile = open("gps.geojson", "w")
    jsonFile.writelines('{\n"type": "FeatureCollection","features": [\n')

    def write_data(paths):
        index = 1
        for path in all_files('./' + paths, '*.jpg'):
            f = open(path[2:], 'rb')
            tags = exifread.process_file(f)
            # jsonFile.writelines('"type": "Feature","properties": {"cartodb_id":"'+str(index)+'"},"geometry": {"type": "Point","coordinates": [')
            latitude = tags['GPS GPSLatitude'].printable[1:-1]
            longitude = tags['GPS GPSLongitude'].printable[1:-1]
            print latitude
            print parse_gps(latitude)
            # print tags['GPS GPSLongitudeRef']
            # print tags['GPS GPSLatitudeRef']
            jsonFile.writelines('{"type": "Feature","properties": {"cartodb_id":"' + str(index) + '"')
            jsonFile.writelines(',"OS":"' + str(tags['Image Software']) + '","Model":"' + str(tags['Image Model']) + '","Picture":"'+str(path[7:])+'"')
            jsonFile.writelines('},"geometry": {"type": "Point","coordinates": [' + str(parse_gps(longitude)) + ',' + str(
                parse_gps(latitude)) + ']}},\n')
            index += 1

    write_data('imgs')

    jsonFile.writelines(']}\n')
    jsonFile.close()

最终代码可见[python cartodb][3]
[3]:https://github.com/gmszone/py_cartodb.git

###上传到cartodb###
