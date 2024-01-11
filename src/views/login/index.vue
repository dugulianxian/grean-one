<script setup lang="ts">
import { ref, reactive, computed, onMounted, onBeforeUnmount } from "vue";
import Motion from "./utils/motion";
import { useRouter } from "vue-router";
import { message } from "@/utils/message";
import { loginRules } from "./utils/rule";
import qrCode from "./components/qrCode.vue";
import update from "./components/update.vue";
import type { FormInstance } from "element-plus";
import { $t, transformI18n } from "@/plugins/i18n";
import { useLayout } from "@/layout/hooks/useLayout";
import { useUserStoreHook } from "@/store/modules/user";
import { initRouter, getTopMenu } from "@/router/utils";
import {
  bg,
  loginSide,
  loginText,
  qrCodeIcon,
  qrPhoneIcon
} from "./utils/static";
import { useRenderIcon } from "@/components/ReIcon/src/hooks";

import Lock from "@iconify-icons/ri/lock-fill";
import User from "@iconify-icons/ri/user-3-fill";

defineOptions({
  name: "Login"
});

const router = useRouter();
const loading = ref(false);
const ruleFormRef = ref<FormInstance>();
const currentPage = computed(() => {
  return useUserStoreHook().currentPage;
});

const { initStorage } = useLayout();
initStorage();

const ruleForm = reactive({
  username: "",
  password: ""
});

const onLogin = async (formEl: FormInstance | undefined) => {
  loading.value = true;
  if (!formEl) return;
  await formEl.validate((valid, fields) => {
    if (valid) {
      useUserStoreHook()
        .loginByUsername({ username: ruleForm.username, password: "admin123" })
        .then(res => {
          if (res.success) {
            // 获取后端路由
            initRouter().then(() => {
              router.push(getTopMenu(true).path);
              message("登录成功", { type: "success" });
            });
          }
        })
        .finally(() => (loading.value = false));
    } else {
      loading.value = false;
      return fields;
    }
  });
};
const handleSignTypeChange = () => {
  if (currentPage.value === 0) {
    useUserStoreHook().SET_CURRENTPAGE(1);
  } else if (currentPage.value === 1) {
    useUserStoreHook().SET_CURRENTPAGE(0);
  } else if (currentPage.value === 2) {
    useUserStoreHook().SET_CURRENTPAGE(0);
  }
};
/** 使用公共函数，避免`removeEventListener`失效 */
function onkeypress({ code }: KeyboardEvent) {
  if (code === "Enter") {
    onLogin(ruleFormRef.value);
  }
}

onMounted(() => {
  window.document.addEventListener("keypress", onkeypress);
});

onBeforeUnmount(() => {
  window.document.removeEventListener("keypress", onkeypress);
});
</script>

<template>
  <div class="select-none">
    <img :src="bg" class="wave" />
    <div class="login-container">
      <div class="img">
        <img :src="loginText" class="img-text" />
        <img :src="loginSide" class="img-side" />
      </div>
      <div class="login-box">
        <div class="login-form">
          <h2 v-if="currentPage === 0 || currentPage === 2" class="title">
            密码登录
          </h2>
          <h2 v-if="currentPage === 1" class="title">扫码登录</h2>
          <el-form
            v-if="currentPage === 0"
            ref="ruleFormRef"
            :model="ruleForm"
            :rules="loginRules"
            size="large"
          >
            <Motion :delay="100">
              <el-form-item prop="username">
                <el-input
                  v-model="ruleForm.username"
                  clearable
                  :placeholder="transformI18n($t('login.username'))"
                  :prefix-icon="useRenderIcon(User)"
                />
              </el-form-item>
            </Motion>

            <Motion :delay="150">
              <el-form-item prop="password">
                <el-input
                  v-model="ruleForm.password"
                  clearable
                  show-password
                  :placeholder="transformI18n($t('login.password'))"
                  :prefix-icon="useRenderIcon(Lock)"
                />
              </el-form-item>
            </Motion>

            <Motion :delay="250">
              <el-form-item>
                <el-button
                  link
                  type="primary"
                  class="forget-btn"
                  @click="useUserStoreHook().SET_CURRENTPAGE(2)"
                >
                  {{ transformI18n($t("login.forget")) }}
                </el-button>
              </el-form-item>
            </Motion>

            <Motion :delay="300">
              <el-form-item>
                <el-button
                  class="login-btn"
                  size="large"
                  type="primary"
                  :loading="loading"
                  @click="onLogin(ruleFormRef)"
                >
                  {{ transformI18n($t("login.login")) }}
                </el-button>
              </el-form-item>
            </Motion>
          </el-form>
          <!-- 二维码登录 -->
          <qrCode v-if="currentPage === 1" />
          <!-- 忘记密码 -->
          <update v-if="currentPage === 2" />
          <!-- 切换登录 -->
          <div class="single-tab" @click="handleSignTypeChange">
            <img v-if="currentPage === 0" alt="" :src="qrCodeIcon" />
            <img v-if="currentPage === 1" alt="" :src="qrPhoneIcon" />
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
@import url("@/style/login.css");
</style>

<style lang="scss" scoped>
:deep(.el-input-group__append, .el-input-group__prepend) {
  padding: 0;
}

.translation {
  ::v-deep(.el-dropdown-menu__item) {
    padding: 5px 40px;
  }

  .check-zh {
    position: absolute;
    left: 20px;
  }

  .check-en {
    position: absolute;
    left: 20px;
  }
}
</style>
