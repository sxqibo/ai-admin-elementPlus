<script setup lang="ts">
/**
 * 一：依赖部分
 */
import { reactive } from 'vue' // 引用vue3
import { Promotion } from '@element-plus/icons-vue' // 引用图标库
import { useI18n } from 'vue-i18n' // 引用国际化
import { CheckState } from '/@/stores/interface/index' // 引用状态接口
import { useCommonStore } from '/@/stores/common' // 引用公共store
import { getEnvPhp, getEnvNpm } from '/@/api/install/index' // 引入安装接口
/**
 * 二：定义部分
 */
// dialog组件自动弹出
const { t, locale } = useI18n() // 国际化

const common = useCommonStore() // 公共store

// 状态设置
const state: CheckState = reactive({
    // 开始表单
    startForm: {
        lang: locale.value, // 语言：默认中文简体
        packageManager: 'pnpm', // NPM包管理器：默认pnpm
        setNpmRegistery: 'taobao', // 设置NPM源：默认淘宝
    },

    // 环境检测数据
    envCheckPhpData: [], // PHP环境检测数据
    envCheckNpmData: [], // NPM环境检测数据

    // 是否显示环境检测
    checkDoneIndex: 'executing', // 三种状态：执行 executing 、成功 ok 、失败 fail
})

/**
 * 三：函数部分
 */
// 语言切换
const changeLang = (val: string) => {
    window.localStorage.setItem('ta-lang', val)
    location.reload()
}

// 改变包管理器
const changePackageManager = () => {
    console.log('改变包管理器 且 下一步操作')
}

// 开始安装
const startInstall = () => {
    // step1:判断显示dialog为true时显示包管理器
    if (common.showStartDialog) {
        changePackageManager()
    }
    // step2:关闭dialog
    common.toggleStartDialog(false)

    // step3:获取PHP环境检测数据
    const packageManagerParam = state.startForm.packageManager
    
    getEnvPhp().then((res: any) => {
        console.log('res_php:', res)
        if (res.data.code == 1) {
            state.envCheckPhpData = res.data.data

            // step4:获取NPM环境检测数据
            getEnvNpm(packageManagerParam).then((res: any) => {
                console.log('res_npm:', res)
                if (res.data.code == 1) {
                    state.envCheckNpmData = res.data.data
                }
            })
        }
    })
}

// 根据文字显示相应的图片
const getSrc = (name: string) => {
    return `/@/assets/install/${name}.png`
}

/**
 * 下一步配置数据库
 */
const goConfig = () => {
    state.checkDoneIndex = 'ok'
    common.setStep('config')
}
</script>

<template>
    <!-- 列表页面/start -->
    <div class="container">
        <div class="table-title">{{ t('Environmental inspection') }}</div>
        <div class="table">

            <!-- 第一步：PHP检测环境 -->
            <div v-for="(item, idx) in state.envCheckPhpData" :key="idx">
                <div class="table-item">
                    <!-- 正常显示内容 -->
                    <div class="table-label">
                        <span>{{ item.name }}</span>
                    </div>
                    <!-- 异常显示内容 -->
                    <div class="table-value">
                        {{ item.describe }}
                        <img title="图片" class="data-state" :src="getSrc(item.state)" :alt="item.state" />
                    </div>
                </div>
            </div>

            <!-- 第二步：NPM检测环境-->
            <div v-for="(item, idx) in state.envCheckNpmData" :key="idx">
                <div class="table-item">
                    <!-- 正常显示内容 -->
                    <div class="table-label">
                        <span>{{ item.name }}</span>
                    </div>
                    <!-- 异常显示内容 -->
                    <div class="table-value">
                        {{ item.describe }}
                        <img title="图片" class="data-state" :src="getSrc(item.state)" :alt="item.state" />
                    </div>
                </div>
            </div>

            <div class="table-item">
                <div class="table-label">
                    <span>是否测试命令执行？</span>
                </div>
                <div class="table-value">
                    <img title="图片" class="data-state" src="/@/assets/install/warn.png" />
                </div>
            </div>

            <div class="table-item">
                <div class="table-label">{{ t('Checking installation environment') }}</div>
                <div class="table-value">
                    <img title="正在检测" class="data-state" alt="正在检测" src="/@/assets/install/loading.gif" />
                </div>
            </div>

            <div class="check-done ok">恭喜，安装可以继续</div>
            <div class="button pass" @click="goConfig">第二点：站点配置</div>
        </div>
    </div>
    <!-- 列表页面/end -->

    <!-- 显示dialog组件/start -->
    <el-dialog
        v-model="common.showStartDialog"
        :title="t('Ready to start')"
        center
        :show-close="false"
        :close-on-click-modal="false"
        :close-on-press-escape="false"
    >
        <!-- 表单内容/start -->
        <el-form label-width="120px" label-position="left" :model="state.startForm" @click="startInstall">
            <!-- 语言 -->
            <el-form-item :label="t('language')">
                <el-select @change="changeLang" class="w100" v-model="state.startForm.lang">
                    <el-option label="中文简体" value="zh-cn"></el-option>
                    <el-option label="English" value="en"></el-option>
                </el-select>
            </el-form-item>

            <!-- NPM包管理器 -->
            <el-form-item :label="t('NPM package manager')"> {{ state.startForm.packageManager }}
                <el-select class="w100" v-model="state.startForm.packageManager">
                    <el-option label="npm" value="npm"></el-option>
                    <el-option label="cnpm" value="cnpm"></el-option>
                    <el-option :label="'pnpm' + t('Recommand')" value="pnpm"></el-option>
                    <el-option :label="'yarn' + t('Recommand')" value="yarn"></el-option>
                    <el-option label="ni" value="ni"></el-option>
                    <el-option :label="t('I want to execute the command manually')" value="none"></el-option>
                </el-select>
            </el-form-item>

            <!-- 设置NPM源 -->
            <el-form-item :label="t('Set NPM source')" v-model="state.startForm.setNpmRegistery">
                <el-radio-group v-model="state.startForm.setNpmRegistery">
                    <el-radio label="none">{{ t('Use current source') }}</el-radio>
                    <el-radio label="taobao">{{ t('TaoBao') }}</el-radio>
                    <el-radio label="npm">NPM</el-radio>
                    <el-radio label="rednpm">RedNPM</el-radio>
                </el-radio-group>
            </el-form-item>
        </el-form>
        <!-- 表单内容/end -->

        <!-- 底部 -->
        <template #footer>
            <span class="dialog-footer">
                <el-button type="primary" size="large" round @click="startInstall">
                    <el-icon><Promotion /></el-icon>
                    <span class="start-install">{{ t('Start installation') }}</span>
                </el-button>
            </span>
        </template>
    </el-dialog>
    <!-- 显示dialog组件/end -->
</template>

<style scoped lang="scss">
// 列表页面/start
.container {
    margin-top: 20px;

    .table-title {
        display: block;
        text-align: center;
        font-size: 20px;
        color: #303133;
    }

    .table {
        max-width: 560px;
        padding: 20px;
        margin: 10px auto;
    }

    .global-warning {
        margin-bottom: 10px;
    }

    .table-item {
        color: #303133;
        display: flex;
        align-items: center;
        justify-content: space-between;
        background-color: #fff;
        padding: 13px 15px;
        margin-bottom: 2px;
        border-radius: 5px;
        transition: all 0.2s ease;

        &:hover {
            background-color: #fcfcfc;
        }

        .table-label {
            font-size: 15px;
        }

        .label-need {
            font-size: 12px;
            color: #f56c6c;
            cursor: pointer;
            padding: 0 4px;
        }

        .label-need.faq,
        .label-need.install-cnpm {
            color: #3f6ad8;

            &:hover {
                text-decoration: underline;
            }
        }

        .label-need.text {
            cursor: text;
        }
    }

    .table-item.error {
        background-color: #f56c6c;
        color: #fff;
    }

    .table-item.success {
        background-color: #67c23a;
        color: #fff;
    }

    .table-value {
        font-size: 13px;
        display: flex;
        align-items: center;
    }

    .data-state {
        width: 20px;
        height: 20px;
        user-select: none;
        margin-left: 5px;
    }
}

// 列表页面/end

// 按钮 / start
.check-done {
    font-size: 14px;
    margin-top: 20px;
    text-align: right;
}

.check-done.ok {
    color: #67c23a;
}

.check-done.fail {
    color: #f56c6c;
}

.button {
    padding: 15px;
    text-align: center;
    font-size: 16px;
    background-color: #409eff;
    border-radius: 5px;
    color: #fff;
    margin-top: 20px;
    opacity: 0.4;
    cursor: pointer;
    transition: all 0.2s ease;
}

.button.pass {
    opacity: 1;
}

// 按钮 / end
.w100 {
    width: 100%;
}

.start-install {
    margin-left: 10px;
}
</style>
