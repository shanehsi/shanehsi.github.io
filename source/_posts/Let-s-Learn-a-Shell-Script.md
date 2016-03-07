title: 学一个 shell 脚本
date: 2016-03-07 15:45:44
tags:
---

```shell
#!/bin/bash
# ------------------
# 线上静态部署脚本
#
# @history
#
# ------------------

WORKSPACE=`pwd`
DIRNAME=`dirname $0`    # 返回这个脚本文件放置的目录，然后可以根据这个目录定位要运行程序的相对位置

STATIC_PATH=/opt/xoxo/static/
TEST_STATIC_PATH=/opt/xoxo/static/test
PROJECT_PATH=$WORKSPACE/xxyy-container
DEPLOY_PATH=$PROJECT_PATH/src/main/webapp/deploy/

pwd;
echo $DIRNAME;  # 打印当前目录

chmod u+x $DIRNAME/color.sh && source $DIRNAME/color.sh     # 给 color.sh 增加执行权限，并内联
if [ $? -eq 0 ]; then   # $? 上个命令的退出状态，或函数的返回值。
    echo -e "${Gre}执行color.sh成功${RCol}"
else
    echo "执行color.sh出错了"
    exit 1
fi

cd $DIRNAME && pwd

npm install --registry=http://r.npm.xoxo.com     # npm install，并使用 xoxo 源，当然这里的代码不依赖 xoxo 源，这句话可能会拖慢速度（比如点评 CI 无法访问 xoxo 源）

if [ $? -eq 0 ]; then
    echo -e "${Gre}npm依赖包加载完成${RCol}"
else
    echo -e "${On_Red}npm依赖包加载失败${RCol}"
    exit 1;
fi

# 函数，调用 grunt deploy
grunt_deploy () {
    # grunt task 的参数 --target，--project_path
    grunt deploy --target=$1 --project_path=$PROJECT_PATH

    if [ $? -eq 0 ]; then
        echo -e "${Gre}$1前端编译成功${RCol}"
    else
        echo -e "${On_Red}执行失败。。。${RCol}"
        echo "将重新编译$1"
        echo -e "\n\n\n\n";
        # 重新编译一次，并 verbose 打印
        grunt deploy --target=$1 --project_path=$PROJECT_PATH --verbose
        echo "----------------------"
        echo "$1"
        exit 1;
    fi
}

# 函数调用，在函数实现中通过 $1 $2 ... $9 来接受函数调用时的变量
grunt_deploy xxyy

echo -e '${Gre}拷贝静态资源到静态服务器${RCol}'

rsync_static () {
    rsync -avi $DEPLOY_PATH xoxo@mobile-static0$1:$STATIC_PATH

    if [ $? -eq 0 ]; then
        echo -e "${Gre}rsync 到 mobile-static0$1 成功${RCol}"
    else
        echo -e "${On_Red}rsync 到 mobile-static0$1 失败${RCol}"
        exit 1
    fi
}

rsync_yf_static () {
    # -a, --archive 归档模式，表示以递归方式传输文件，并保持所有文件属性，等于-rlptgoD 
    # -a, --archive               archive mode; same as -rlptgoD (no -H)
    # -v, --verbose 详细模式输出 
    # -i, --itemize-changes       output a change-summary for all updates
    # 这是 ssh 通道传入，用户名 xoxo 地址 yf-mobile-static0$1（即，yf-mobile-static0xxyy）
    # 从 $DEPLOY_PATH -> 远程机器上的 $STATIC_PATH
    rsync -avi $DEPLOY_PATH xoxo@yf-mobile-static0$1:$STATIC_PATH

    if [ $? -eq 0 ]; then
        echo -e "${Gre}rsync 到 yf-mobile-static0$1 成功${RCol}"
    else
        echo -e "${On_Red}rsync 到 yf-mobile-static0$1 失败${RCol}"
        exit 1
    fi
}

# rsync_static 1
# rsync_static 2

rsync_yf_static 1
rsync_yf_static 2

# 清空发布后的静态文件
rm -rf $DEPLOY_PATH
```
