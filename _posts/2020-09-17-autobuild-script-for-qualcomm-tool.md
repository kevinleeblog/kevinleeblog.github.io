---
layout: post
title: "Autobuild script for Qualcomm tool"
auther: Kevin Lee
category: project1
tags: [Android Open Source Project, MSM8953, Qualcomm]
subtitle:
visualworkflow: true
---

Aosp使用高通CPU都不免會需要了解使用Qfil燒錄、OTA燒錄以及adb燒錄等，這三種燒錄產生的image方式都不一樣，使用aosp所編譯出來的就是最原始的image可直接使用adb方式來燒錄

但是在空板時，還是需要先用QFIL燒錄一次，之後在用OTA或是adb方式來更新，所以從事Android BSP的工作需要一次產生QFIL、OTA、adb用的image來應付各式不同狀況，如果有一個script可以編譯完成後自動做這些事情，那該有多美好！！

於是本人就創作出這段shell script

直接po出完成品

*easybuild.sh*

```shell
#!/bin/bash
TARGET_PRODUCT="msm8953_64"
SHELL_NAME=$0
TARGET_BUILD_VARIANT=$1
TARGET_BUILD_SKULL=$2
PREBUILD_TOOL_NAME="SC600_Android_9.0.0_prebuilt_for_QFIL_20190409"
LOG_FILE_NAME="build"
BUILD_DATE=$(date +%Y%m%d_%H%M)
CPU_CORE_NUM=$((`grep -c ^processor /proc/cpuinfo`))
TOOLS_PATH="build/image"
IMG_OUT_PATH="out"
IMG_OUT_NAME="out_img"
BUILD_DIR="${IMG_OUT_PATH}/${IMG_OUT_NAME}"
AOSP_OUTPUT_IMAGE_PATH="${IMG_OUT_PATH}/target/product/msm8953_64"

function usage(){
    echo -e "Usage:\n\t${SHELL_NAME} [<user|userdebug|eng>] [YEM|YWF|YNA|YJP]"
    exit -1
}

if [ -z "$TARGET_BUILD_VARIANT" ] ; then
    usage
fi

if [ ${TARGET_BUILD_VARIANT} != "user" ] && [ ${TARGET_BUILD_VARIANT} != "userdebug" ] \
&& [ ${TARGET_BUILD_VARIANT} != "eng" ] ; then
    echo -e "Android Build-Type \"${TARGET_BUILD_VARIANT}\" is wrong!"
    usage
fi

if [ -z "$TARGET_BUILD_SKULL" ] ; then
    TARGET_BUILD_SKULL="YEM"
fi

case "${TARGET_BUILD_SKULL}" in
    "YEM" )
        export SKULL=42
    ;;
    "YWF" )
        export SKULL=41
    ;;
    "YNA" )
        export SKULL=43
    ;;
    "YJP" )
        export SKULL=44
    ;;
    * )
        echo -e "Skull module \"${TARGET_BUILD_SKULL}\" is wrong!"
        usage
    ;;
esac

BUILD_NUMBER_DIR_NAME="V7A-E614-Z00-${SKULL}-`date +%Y%m%d|sed 's/^..//'`"
RAW_IMAGE_DIR="${BUILD_NUMBER_DIR_NAME}_RAW_image"
RAW_IMAGE_PATH="${BUILD_DIR}/${RAW_IMAGE_DIR}"
OTA_IMAGE_DIR="${BUILD_NUMBER_DIR_NAME}_OTA_image"
QFIL_IMAGE_DIR="${BUILD_NUMBER_DIR_NAME}_QFIL_image"
QFIL_FLAT_BUILD_DIR_NAME="${QFIL_IMAGE_DIR}"
QFIL_IMAGE_PATH="${BUILD_DIR}/${QFIL_FLAT_BUILD_DIR_NAME}/emmc"

function packing_raw_image(){
    local RAW_IMG_LIST=( "boot.img" "cache.img" "dtbo.img" "emmc_appsboot.mbn" \
    "mdtp.img" "persist.img" "recovery.img" "system.img" "userdata.img" \
    "vbmeta.img" "vendor.img" )

    if [ ! -d ${RAW_IMAGE_PATH} ] ; then
        mkdir -p ${RAW_IMAGE_PATH}
    fi

    for ((i=0; i < ${#RAW_IMG_LIST[@]}; i++))
    do
        cp -rv ${AOSP_OUTPUT_IMAGE_PATH}/${RAW_IMG_LIST[$i]} ${RAW_IMAGE_PATH}
        if [ ! $? = "0" ] ; then
            echo -e "Copy \"${RAW_IMG_LIST[$i]}\" to ${RAW_IMAGE_PATH} error"
            exit -1
        fi
    done

    cp -r ${TOOLS_PATH}/splash.img ${RAW_IMAGE_PATH}
    cp -r ${TOOLS_PATH}/flash_image.sh ${RAW_IMAGE_PATH}

    case "${SKULL}" in
    "42" )
        cp -rv ${TOOLS_PATH}/modem_img/SC600YEMPAR05A04/NON-HLOS.bin ${RAW_IMAGE_PATH}
    ;;
    "41" )
        cp -rv ${TOOLS_PATH}/modem_img/SC600YWFPAR05A04/NON-HLOS.bin ${RAW_IMAGE_PATH}
    ;;
    "43" )
        cp -rv ${TOOLS_PATH}/modem_img/SC600YNAPAR05A04/NON-HLOS.bin ${RAW_IMAGE_PATH}
    ;;
    "44" )
        cp -rv ${TOOLS_PATH}/modem_img/SC600YJPPAR05A06/NON-HLOS.bin ${RAW_IMAGE_PATH}
    ;;
    esac
    zip ${BUILD_DIR}/${RAW_IMAGE_DIR} ${RAW_IMAGE_PATH}/* 
}

function prebuild_with_qfil(){
    echo -e "Copy and unzip prebuild tool\n"
    sleep 3
    unzip ${TOOLS_PATH}/qfil/${PREBUILD_TOOL_NAME}.zip -d ${BUILD_DIR}
    case "${SKULL}" in
    "42" )
        unzip ${TOOLS_PATH}/qfil/SC600YEMPAR05A04_BP01.004_prebuilt.zip -d ${BUILD_DIR}
        cp -rv ${BUILD_DIR}/SC600YEMPAR05A04_BP01.004_prebuilt/* ${BUILD_DIR}/${PREBUILD_TOOL_NAME}
    ;;
    "41" )
        unzip ${TOOLS_PATH}/qfil/SC600YWFPAR05A04_BP01.004_prebuilt.zip -d ${BUILD_DIR}
        cp -rv ${BUILD_DIR}/SC600YWFPAR05A04_BP01.004_prebuilt/* ${BUILD_DIR}/${PREBUILD_TOOL_NAME}
    ;;
    "43" )
        unzip ${TOOLS_PATH}/qfil/SC600YNAPAR05A04_BP01.004_prebuilt.zip -d ${BUILD_DIR}
        cp -rv ${BUILD_DIR}/SC600YNAPAR05A04_BP01.004_prebuilt/* ${BUILD_DIR}/${PREBUILD_TOOL_NAME}
    ;;
    "44" )
        unzip ${TOOLS_PATH}/qfil/SC600YJPPAR05A06_BP01.006_prebuilt.zip -d ${BUILD_DIR}
        cp -rv ${BUILD_DIR}/SC600YJPPAR05A06_BP01.006_prebuilt.zip/* ${BUILD_DIR}/${PREBUILD_TOOL_NAME}
    ;;
    esac

    cp -rv ${RAW_IMAGE_PATH}/* \
    ${BUILD_DIR}/${PREBUILD_TOOL_NAME}/LA.UM.7.6.2/LINUX/android/out/target/product/msm8953_64/

    cd ${BUILD_DIR}/${PREBUILD_TOOL_NAME}/MSM8953.LA.3.2.1/common/build/
    /usr/bin/python build.py
    cd -

    cp -r ${TOOLS_PATH}/fh_loader.exe ${BUILD_DIR}
    cd ${BUILD_DIR}
    /usr/bin/wine fh_loader.exe --search_path=${PREBUILD_TOOL_NAME} --contentsxml="${PREBUILD_TOOL_NAME}/contents.xml" \
    --flavor=asic --flattenbuildto="${QFIL_FLAT_BUILD_DIR_NAME}" --noprompt --showpercentagecomplete --zlpawarehost=1 --memoryname=emmc
    cd -
 
    sed -i '5s/""/"NON-HLOS.bin"/' ${QFIL_IMAGE_PATH}/rawprogram_unsparse.xml
    sed -i '22s/""/"persist.img"/' ${QFIL_IMAGE_PATH}/rawprogram_unsparse.xml
    sed -i '23s/""/"splash.img"/' ${QFIL_IMAGE_PATH}/rawprogram_unsparse.xml
    local RAW_IMG_LIST=( "persist.img" "splash.img" "NON-HLOS.bin" )
    for ((i=0; i < ${#RAW_IMG_LIST[@]}; i++))
    do
        cp -rv ${RAW_IMAGE_PATH}/${RAW_IMG_LIST[$i]} ${QFIL_IMAGE_PATH}
        if [ ! $? = "0" ] ; then
            echo -e "Copy \"${RAW_IMG_LIST[$i]}\" to ${QFIL_IMAGE_PATH} error"
            exit -1
        fi
    done
    
    cd ${BUILD_DIR}/${QFIL_FLAT_BUILD_DIR_NAME}
    zip ${QFIL_IMAGE_DIR} emmc/*
    mv *.zip ../
    cd -
}

if [ ! -d ${BUILD_DIR} ] ; then
    echo -e "Create ${BUILD_DIR} folder"
    mkdir -p ${BUILD_DIR}
else
    echo -e "Remove ${BUILD_DIR} folder"
    rm -rf ${BUILD_DIR}
    mkdir -p ${BUILD_DIR}
fi

echo -e "TARGET_PRODUCT = ${TARGET_PRODUCT}"
echo -e "TARGET_BUILD_VARIANT = ${TARGET_BUILD_VARIANT}"
echo -e "TARGET_BUILD_SKULL = SC600_${TARGET_BUILD_SKULL}"
echo -e "CPU Cores = ${CPU_CORE_NUM}"
echo -e "AOSP start building ....\n"
sleep 3

(source build/envsetup.sh && lunch ${TARGET_PRODUCT}-${TARGET_BUILD_VARIANT} && make -j ${CPU_CORE_NUM} ) | tee ${BUILD_DIR}/${LOG_FILE_NAME}_${BUILD_DATE}.log
if [ $? = "0" ] ; then
    echo -e "AOSP build success!\n"
    
    if [ -d ${AOSP_OUTPUT_IMAGE_PATH} ] ; then
        echo -e "Packing RAW image..."
	    sleep 3
        packing_raw_image
        prebuild_with_qfil

        ### OTA ###
        echo -e "Generate OTA...\n"
        sleep 3
        (source build/envsetup.sh && lunch ${TARGET_PRODUCT}-${TARGET_BUILD_VARIANT} && make -j ${CPU_CORE_NUM} dist DIST_DIR=${BUILD_DIR})
        ./build/make/tools/releasetools/ota_from_target_files ${BUILD_DIR}/${TARGET_PRODUCT}-target_files.zip ${BUILD_DIR}/${OTA_IMAGE_DIR}.zip
    fi
else
    echo -e "AOSP build failure!\n"
    exit -1    
fi

```

使用方式

easybuild.sh放在aosp的根目錄下
easybuild.sh [<user|userdebug|eng>] [YEM|YWF|YNA|YJP]

./easybuild.sh user YEM表示要產生SC600_YEM模組用的image