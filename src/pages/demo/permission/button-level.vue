<script lang="ts" setup>
import { checkPermission } from "@@/utils/permission"
import {
  ArrowLeft,
  CirclePlus,
  DataLine,
  Lock,
  Refresh,
  User,
  Warning
} from "@element-plus/icons-vue"
import { useRouter } from "vue-router"
import SwitchRoles from "./components/SwitchRoles.vue"

defineOptions({
  name: "PermissionButtonLevel"
})

const router = useRouter()

type PageStatus = "default" | "loading" | "empty" | "error" | "forbidden"

const currentStatus = ref<PageStatus>("default")

const statusOptions = [
  { value: "default", label: "正常状态", icon: DataLine },
  { value: "loading", label: "加载中", icon: Refresh },
  { value: "empty", label: "空数据", icon: DataLine },
  { value: "error", label: "加载失败", icon: Warning },
  { value: "forbidden", label: "无权限", icon: Lock }
]

const errorMessage = ref("服务器连接超时，请稍后重试")
const retryCount = ref(0)
const maxRetries = 3

function handleRetry() {
  if (retryCount.value >= maxRetries) {
    ElMessage.error(`已重试 ${maxRetries} 次，请稍后再试或联系技术支持`)
    return
  }
  retryCount.value++
  currentStatus.value = "loading"
  setTimeout(() => {
    if (Math.random() > 0.5) {
      currentStatus.value = "default"
      retryCount.value = 0
      ElMessage.success("数据加载成功")
    } else {
      currentStatus.value = "error"
      errorMessage.value = retryCount.value >= maxRetries
        ? "多次重试失败，请检查网络连接或联系技术支持"
        : "数据加载失败，点击重试"
    }
  }, 1500)
}

function handleGoBack() {
  if (window.history.length > 1) {
    router.back()
  } else {
    router.push("/")
    ElMessage.info("没有历史记录，已跳转到首页")
  }
}

function handleContactAdmin() {
  ElMessageBox.confirm(
    "请联系管理员开通权限\n\n管理员邮箱: admin@example.com\n联系电话: 138-xxxx-xxxx",
    "联系管理员",
    {
      confirmButtonText: "知道了",
      cancelButtonText: "取消",
      type: "info",
      showCancelButton: false
    }
  )
}

function handleRefresh() {
  currentStatus.value = "loading"
  setTimeout(() => {
    currentStatus.value = "default"
    ElMessage.success("刷新成功")
  }, 1000)
}

function handleCreateData() {
  ElMessage.info("跳转到数据创建页面...")
}

const mockTableData = ref([
  { id: 1, name: "用户管理", description: "管理系统用户", status: "active" },
  { id: 2, name: "角色管理", description: "管理用户角色", status: "active" },
  { id: 3, name: "权限配置", description: "配置系统权限", status: "inactive" }
])
</script>

<template>
  <div class="app-container">
    <SwitchRoles />

    <el-card shadow="never" class="status-switcher-card">
      <template #header>
        <div class="card-header">
          <span class="title">状态演示</span>
          <span class="subtitle">切换下方按钮查看不同状态的页面表现</span>
        </div>
      </template>
      <div class="status-buttons">
        <el-radio-group v-model="currentStatus" size="large">
          <el-radio-button
            v-for="status in statusOptions"
            :key="status.value"
            :value="status.value"
          >
            <el-icon class="mr-1">
              <component :is="status.icon" />
            </el-icon>
            {{ status.label }}
          </el-radio-button>
        </el-radio-group>
      </div>
    </el-card>

    <div class="content-area">
      <Transition name="fade" mode="out-in">
        <div v-if="currentStatus === 'loading'" key="loading">
          <div class="status-wrapper loading-wrapper">
            <el-card shadow="never">
              <template #header>
                <el-skeleton animated :rows="1" />
              </template>
              <div class="loading-content">
                <div class="loading-animation">
                  <el-icon class="loading-icon">
                    <Refresh />
                  </el-icon>
                </div>
                <div class="loading-text">
                  <h3>数据加载中</h3>
                  <p>正在为您获取最新数据，请稍候...</p>
                </div>
                <div class="loading-skeleton">
                  <el-skeleton animated :rows="4" />
                </div>
              </div>
            </el-card>
          </div>
        </div>

        <div v-else-if="currentStatus === 'empty'" key="empty">
          <div class="status-wrapper empty-wrapper">
            <el-card shadow="never">
              <template #header>
                <div class="card-header">
                  <span class="title">权限列表</span>
                  <el-button type="primary" :icon="Refresh" circle @click="handleRefresh" />
                </div>
              </template>
              <div class="empty-content">
                <el-empty description="">
                  <template #image>
                    <div class="empty-icon-wrapper">
                      <el-icon class="empty-icon">
                        <DataLine />
                      </el-icon>
                    </div>
                  </template>
                  <h3>暂无数据</h3>
                  <p class="empty-desc">
                    当前还没有任何权限配置记录
                  </p>
                  <p class="empty-suggestion">
                    您可以：
                  </p>
                  <div class="empty-actions">
                    <el-button type="primary" :icon="CirclePlus" @click="handleCreateData">
                      新增权限
                    </el-button>
                    <el-button :icon="Refresh" @click="handleRefresh">
                      刷新页面
                    </el-button>
                  </div>
                </el-empty>
              </div>
            </el-card>
          </div>
        </div>

        <div v-else-if="currentStatus === 'error'" key="error">
          <div class="status-wrapper error-wrapper">
            <el-card shadow="never">
              <template #header>
                <div class="card-header">
                  <span class="title">权限列表</span>
                  <el-button type="primary" :icon="Refresh" circle @click="handleRetry" />
                </div>
              </template>
              <div class="error-content">
                <div class="error-icon-wrapper">
                  <el-icon class="error-icon">
                    <Warning />
                  </el-icon>
                </div>
                <h3>加载失败</h3>
                <p class="error-message">
                  {{ errorMessage }}
                </p>
                <div class="error-details" v-if="retryCount > 0">
                  <el-tag size="small" type="warning">
                    已重试 {{ retryCount }}/{{ maxRetries }} 次
                  </el-tag>
                </div>
                <div class="error-actions">
                  <el-button
                    type="primary"
                    :icon="Refresh"
                    :disabled="retryCount >= maxRetries"
                    :loading="currentStatus === 'loading'"
                    @click="handleRetry"
                  >
                    {{ retryCount >= maxRetries ? "重试次数已用完" : "重新加载" }}
                  </el-button>
                  <el-button :icon="ArrowLeft" @click="handleGoBack">
                    返回上一页
                  </el-button>
                </div>
                <div class="error-suggestion">
                  <el-alert
                    title="您也可以尝试以下方法"
                    type="info"
                    :closable="false"
                    show-icon
                  >
                    <ul class="suggestion-list">
                      <li>检查网络连接是否正常</li>
                      <li>确认服务器状态是否正常</li>
                      <li>稍后再次尝试</li>
                      <li>如问题持续存在，请联系技术支持</li>
                    </ul>
                  </el-alert>
                </div>
              </div>
            </el-card>
          </div>
        </div>

        <div v-else-if="currentStatus === 'forbidden'" key="forbidden">
          <div class="status-wrapper forbidden-wrapper">
            <el-card shadow="never">
              <div class="forbidden-content">
                <div class="forbidden-icon-wrapper">
                  <el-icon class="forbidden-icon">
                    <Lock />
                  </el-icon>
                </div>
                <h3>暂无访问权限</h3>
                <p class="forbidden-desc">
                  抱歉，您当前的角色没有权限访问此页面
                </p>
                <div class="forbidden-info">
                  <el-descriptions :column="1" border size="small">
                    <el-descriptions-item label="当前角色">
                      <el-tag type="info" size="large">
                        <el-icon class="mr-1">
                          <User />
                        </el-icon>
                        editor
                      </el-tag>
                    </el-descriptions-item>
                    <el-descriptions-item label="所需权限">
                      <el-tag type="warning" size="large">
                        admin
                      </el-tag>
                    </el-descriptions-item>
                    <el-descriptions-item label="可访问内容">
                      按钮级权限演示、基础数据查看
                    </el-descriptions-item>
                  </el-descriptions>
                </div>
                <div class="forbidden-actions">
                  <el-button type="primary" :icon="User" @click="handleContactAdmin">
                    联系管理员
                  </el-button>
                  <el-button :icon="ArrowLeft" @click="handleGoBack">
                    返回上一页
                  </el-button>
                </div>
                <div class="forbidden-suggestion">
                  <el-alert
                    title="提示"
                    type="warning"
                    :closable="false"
                    show-icon
                  >
                    <p>您可以通过以下方式获取权限：</p>
                    <ul class="suggestion-list">
                      <li>联系系统管理员申请开通相应权限</li>
                      <li>使用有权限的账号重新登录</li>
                      <li>检查当前登录账号的角色配置</li>
                    </ul>
                  </el-alert>
                </div>
              </div>
            </el-card>
          </div>
        </div>

        <div v-else key="default">
          <el-card header="权限指令 v-permission 示例" shadow="never" class="margin-top-20">
            <el-button v-permission="['admin']" type="primary">
              仅 admin 可见
            </el-button>
            <el-button v-permission="['admin', 'editor']">
              admin 和 editor 都可见
            </el-button>
          </el-card>

          <el-card header="权限函数 checkPermission 示例" shadow="never" class="margin-top-20">
            <el-text type="warning" size="large">
              Element Plus 的 el-tab-pane 和 el-table-column 以及其它动态渲染 DOM 的场景不适合使用 v-permission
              这种情况下你可以通过 v-if + checkPermission 来实现
            </el-text>
            <el-tabs type="border-card" class="margin-top-20">
              <el-tab-pane v-if="checkPermission(['admin'])" label="admin 专属">
                <el-tag type="primary" size="large">
                  此内容仅 admin 可见
                </el-tag>
                <p class="tab-content">
                  这是管理员专属的内容区域，包含敏感配置项。
                </p>
              </el-tab-pane>
              <el-tab-pane v-if="checkPermission(['admin', 'editor'])" label="共同可见">
                <el-tag type="success" size="large">
                  admin 和 editor 都可见
                </el-tag>
                <p class="tab-content">
                  这是公共内容区域，所有具备编辑权限的用户都可以查看。
                </p>
              </el-tab-pane>
            </el-tabs>
          </el-card>

          <el-card shadow="never" class="margin-top-20">
            <template #header>
              <div class="card-header">
                <span class="title">权限列表模拟</span>
                <div>
                  <el-button type="primary" :icon="CirclePlus">
                    新增权限
                  </el-button>
                  <el-button :icon="Refresh" @click="handleRefresh">
                    刷新
                  </el-button>
                </div>
              </div>
            </template>
            <el-table :data="mockTableData" stripe>
              <el-table-column prop="id" label="ID" width="80" align="center" />
              <el-table-column prop="name" label="权限名称" min-width="150" />
              <el-table-column prop="description" label="描述" min-width="200" />
              <el-table-column prop="status" label="状态" width="100" align="center">
                <template #default="scope">
                  <el-tag :type="scope.row.status === 'active' ? 'success' : 'info'" effect="plain">
                    {{ scope.row.status === 'active' ? '启用' : '禁用' }}
                  </el-tag>
                </template>
              </el-table-column>
              <el-table-column label="操作" width="200" align="center" fixed="right">
                <template #default>
                  <el-button type="primary" text bg size="small">
                    编辑
                  </el-button>
                  <el-button type="danger" text bg size="small">
                    删除
                  </el-button>
                </template>
              </el-table-column>
            </el-table>
          </el-card>
        </div>
      </Transition>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.margin-top-20 {
  margin-top: 20px;
}

.status-switcher-card {
  margin-bottom: 20px;

  .card-header {
    display: flex;
    flex-direction: column;
    gap: 4px;

    .title {
      font-size: 16px;
      font-weight: 600;
      color: var(--el-text-color-primary);
    }

    .subtitle {
      font-size: 13px;
      color: var(--el-text-color-secondary);
    }
  }
}

.status-buttons {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.content-area {
  min-height: 400px;
}

.status-wrapper {
  display: flex;
  flex-direction: column;
}

.loading-wrapper {
  .loading-content {
    padding: 40px 20px;
    text-align: center;
  }

  .loading-animation {
    margin-bottom: 24px;
  }

  .loading-icon {
    font-size: 48px;
    color: var(--el-color-primary);
    animation: rotate 1s linear infinite;
  }

  .loading-text {
    h3 {
      margin: 0 0 8px 0;
      font-size: 18px;
      color: var(--el-text-color-primary);
    }

    p {
      margin: 0;
      color: var(--el-text-color-secondary);
      font-size: 14px;
    }
  }

  .loading-skeleton {
    margin-top: 32px;
    max-width: 600px;
    margin-left: auto;
    margin-right: auto;
  }
}

@keyframes rotate {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}

.empty-wrapper {
  .card-header {
    display: flex;
    justify-content: space-between;
    align-items: center;

    .title {
      font-size: 16px;
      font-weight: 600;
    }
  }

  .empty-content {
    padding: 60px 20px;
  }

  .empty-icon-wrapper {
    display: flex;
    justify-content: center;
    margin-bottom: 16px;
  }

  .empty-icon {
    font-size: 64px;
    color: var(--el-text-color-placeholder);
  }

  h3 {
    margin: 0 0 8px 0;
    font-size: 18px;
    color: var(--el-text-color-primary);
  }

  .empty-desc {
    margin: 0 0 16px 0;
    color: var(--el-text-color-secondary);
    font-size: 14px;
  }

  .empty-suggestion {
    margin: 0 0 24px 0;
    color: var(--el-text-color-secondary);
    font-size: 13px;
  }

  .empty-actions {
    display: flex;
    justify-content: center;
    gap: 12px;
    flex-wrap: wrap;
  }
}

.error-wrapper {
  .card-header {
    display: flex;
    justify-content: space-between;
    align-items: center;

    .title {
      font-size: 16px;
      font-weight: 600;
    }
  }

  .error-content {
    padding: 40px 20px;
    text-align: center;
  }

  .error-icon-wrapper {
    margin-bottom: 20px;
  }

  .error-icon {
    font-size: 56px;
    color: var(--el-color-warning);
  }

  h3 {
    margin: 0 0 12px 0;
    font-size: 20px;
    color: var(--el-text-color-primary);
  }

  .error-message {
    margin: 0 0 16px 0;
    color: var(--el-text-color-secondary);
    font-size: 14px;
  }

  .error-details {
    margin-bottom: 24px;
  }

  .error-actions {
    display: flex;
    justify-content: center;
    gap: 12px;
    margin-bottom: 24px;
    flex-wrap: wrap;
  }

  .error-suggestion {
    max-width: 500px;
    margin: 0 auto;
    text-align: left;

    :deep(.el-alert__description) {
      margin-top: 8px;
    }
  }
}

.forbidden-wrapper {
  .forbidden-content {
    padding: 40px 20px;
    text-align: center;
  }

  .forbidden-icon-wrapper {
    margin-bottom: 20px;
  }

  .forbidden-icon {
    font-size: 56px;
    color: var(--el-color-danger);
  }

  h3 {
    margin: 0 0 12px 0;
    font-size: 20px;
    color: var(--el-text-color-primary);
  }

  .forbidden-desc {
    margin: 0 0 24px 0;
    color: var(--el-text-color-secondary);
    font-size: 14px;
  }

  .forbidden-info {
    max-width: 450px;
    margin: 0 auto 24px;

    :deep(.el-descriptions__label) {
      width: 100px;
    }
  }

  .forbidden-actions {
    display: flex;
    justify-content: center;
    gap: 12px;
    margin-bottom: 24px;
    flex-wrap: wrap;
  }

  .forbidden-suggestion {
    max-width: 500px;
    margin: 0 auto;
    text-align: left;

    :deep(.el-alert__description) {
      margin-top: 8px;
    }
  }
}

.suggestion-list {
  margin: 8px 0 0 0;
  padding-left: 20px;

  li {
    margin: 4px 0;
    color: var(--el-text-color-secondary);
    font-size: 13px;
  }
}

.tab-content {
  margin-top: 12px;
  color: var(--el-text-color-secondary);
  font-size: 14px;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;

  .title {
    font-size: 16px;
    font-weight: 600;
  }
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.2s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

@media (max-width: 768px) {
  .status-buttons {
    :deep(.el-radio-group) {
      display: flex;
      flex-wrap: wrap;
    }

    :deep(.el-radio-button__inner) {
      white-space: nowrap;
    }
  }

  .empty-actions,
  .error-actions,
  .forbidden-actions {
    flex-direction: column;
    align-items: stretch;

    .el-button {
      width: 100%;
      justify-content: center;
    }
  }

  .forbidden-info {
    :deep(.el-descriptions) {
      :deep(.el-descriptions__label) {
        width: auto;
      }
    }
  }

  .card-header {
    flex-direction: column;
    align-items: flex-start;
    gap: 12px;

    .title {
      font-size: 15px;
    }

    .el-button {
      width: 100%;
    }
  }

  .loading-text {
    h3 {
      font-size: 16px;
    }

    p {
      font-size: 13px;
    }
  }

  .empty-content,
  .error-content,
  .forbidden-content {
    padding: 30px 16px;
  }

  .empty-icon,
  .error-icon,
  .forbidden-icon,
  .loading-icon {
    font-size: 40px;
  }

  h3 {
    font-size: 16px;
  }

  .empty-desc,
  .error-message,
  .forbidden-desc {
    font-size: 13px;
  }
}

@media (max-width: 480px) {
  .status-switcher-card {
    :deep(.el-card__body) {
      padding: 16px;
    }
  }

  .status-buttons {
    :deep(.el-radio-button__inner) {
      padding: 8px 12px;
      font-size: 13px;
    }
  }

  .suggestion-list {
    li {
      font-size: 12px;
    }
  }
}
</style>
