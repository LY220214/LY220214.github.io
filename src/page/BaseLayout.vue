<template>
  <div style="display: flex;flex-direction: column;align-items:center; justify-content:center;margin-top: 20px">
    <el-form :model="form" label-width="120px">
      <el-form-item label="教师认证">
        <el-input
            v-model="token"
            placeholder="可为空 若需认证请联系作者"
        ></el-input>
      </el-form-item>
      <el-form-item label="前置条件">
        <el-cascader
            placeholder="阶段/科目/年级"
            v-model="previewData"
            :props="props"
            @change="handleChange"
        />
      </el-form-item>
      <el-form-item label="选择分科">
        <el-radio-group v-model="form.studyGroupId" @change="studyGroupChange()">
          <el-radio-button v-for="item in studyGroups" :key="item.studyGroupId" :label="item.studyGroupId">
            <template #default>
              {{ item.studyGroupName }}
            </template>
          </el-radio-button>
        </el-radio-group>
      </el-form-item>
      <el-form-item label="选择毕业年份">
        <el-select v-model="form.entranceYearId" class="m-2" placeholder="毕业年份" @change="entranceYearsChange()">
          <el-option
              v-for="item in options"
              :key="item.value"
              :label="item.label"
              :value="item.value"
          />
        </el-select>
      </el-form-item>
      <el-form-item label="选择学期">
        <el-radio-group v-model="form.term">
          <el-radio-button v-for="item in terms" :key="item.term" :label="item.term">
            <template #default>
              {{ item.termName }}
            </template>
          </el-radio-button>
        </el-radio-group>
      </el-form-item>
      <el-form-item>
        <el-form-item>
          <el-button type="primary" @click="onSubmit">查询资源</el-button>
        </el-form-item>
      </el-form-item>
    </el-form>
    <el-row :gutter="20" style="margin-top: 20px">
      <el-col :span="12" v-show="videos.length > 0">
        <div>视频资源：</div>
        <el-table :data="videos" height="400" style="width: 100%;font-size: small"
                  @selection-change="videosSelectionChange">
          <el-table-column type="selection" width="55"/>
          <el-table-column prop="id" label="编号" width="60"/>
          <el-table-column prop="name" label="名称" width="300"/>
          <el-table-column prop="url" label="地址" width="200" show-overflow-tooltip/>
          <el-table-column align="right">
            <template #header>
              <el-button :icon="Download" circle type="primary" plain v-show="videosSelection.length > 0"
                         @click="downloadVideos"></el-button>
            </template>
          </el-table-column>
        </el-table>
      </el-col>
      <el-col :span="12" v-show="ppts.length > 0">
        <div>文档资源：</div>
        <el-table :data="ppts" height="400" style="width: 100%;font-size: small"
                  @selection-change="pptsSelectionChange">
          <el-table-column type="selection" width="60"/>
          <el-table-column prop="id" label="编号" width="50"/>
          <el-table-column prop="name" label="名称" width="300"/>
          <el-table-column prop="url" label="地址" width="200" show-overflow-tooltip/>
          <el-table-column align="right">
            <template #header>
              <el-button :icon="Download" circle type="primary" plain v-show="pptsSelection.length > 0"
                         @click="downloadPPTs"></el-button>
            </template>
          </el-table-column>
        </el-table>
      </el-col>
    </el-row>
  </div>
</template>
<style scoped>

</style>
<script lang="ts" setup>
import {onMounted, Ref, ref} from 'vue'
import axios from "axios";
import {CascaderProps, ElLoading, ElMessageBox, ElNotification} from "element-plus";
import {Download} from "@element-plus/icons-vue";

onMounted(() => {
  ElMessageBox.alert(
      '<p>欢迎您参与程序测试。在使用本程序之前，请务必仔细阅读以下免责声明条款。通过访问和使用本程序，即表示您同意遵守以下条款和条件：</p>\n' +
      '<div style="height: 300px; overflow-y: scroll">\n' +
      '\n' +
      '  <ol>\n' +
      '    <li>\n' +
      '      <strong>技术讨论：</strong>本程序仅用于技术讨论。未经授权，严禁访问和使用本程序。如果您未购买相应的许可或未获得原网站明确的授权，您不得以任何形式使用本程序。\n' +
      '    </li>\n' +
      '\n' +
      '    <li>\n' +
      '      <strong>个人责任：</strong>使用者对使用本程序所产生的任何结果和后果负有全部责任。本程序的开发者和所有相关方不对因使用本程序而产生的任何损失、损害或纠纷负责。\n' +
      '    </li>\n' +
      '\n' +
      '    <li>\n' +
      '      <strong>免责声明：</strong>本程序按照目前的技术水平和理解编写。我们尽力保证程序的准确性和有效性，但不对程序的功能或性能做出任何明示或暗示的保证。\n' +
      '    </li>\n' +
      '\n' +
      '    <li>\n' +
      '      <strong>拒绝保证：</strong>在适用法律允许的最大范围内，本程序的开发者明确声明不提供任何形式的担保，包括但不限于适销性、特定用途适用性和非侵权性等。您承认使用本程序的风险自负。\n' +
      '    </li>\n' +
      '\n' +
      '    <li>\n' +
      '      <strong>安全性：</strong>尽管我们已经采取了合理的安全措施来保护本程序的安全性，但我们无法保证程序免受黑客攻击、病毒感染、数据泄露或其他未经授权的访问。\n' +
      '    </li>\n' +
      '\n' +
      '    <li>\n' +
      '      <strong>第三方内容：</strong>本程序可能包含来自第三方的内容或链接。这些第三方内容和链接仅供您参考，我们不对其真实性、准确性或合法性负责。您访问和使用这些内容和链接时需要自行判断和承担风险。\n' +
      '    </li>\n' +
      '\n' +
      '    <li>\n' +
      '      <strong>知识产权：</strong>本程序及其相关内容受版权和其他知识产权法律的保护。未经授权，您不得复制、修改、发布、传播或出售本程序或相关内容。\n' +
      '    </li>\n' +
      '\n' +
      '    <li>\n' +
      '      <strong>法律适用：</strong>本免责声明将受到您所在地法律的管辖。任何因与本程序使用相关的纠纷应尽力通过友好协商解决。如果协商无果，双方同意提交有管辖权的法院解决。\n' +
      '    </li>\n' +
      '\n' +
      '    <li>\n' +
      '      <strong>条款修改：</strong>我们保留随时修改本免责声明条款的权利。修改后的条款将在我们的网站上发布，并自发布时生效。建议您定期查阅免责声明以了解任何变更。\n' +
      '    </li>\n' +
      '  </ol>\n' +
      '</div>\n' +
      '<p>通过访问和使用本程序，您确认已充分理解并同意以上免责声明条款。如果您不同意这些条款，请立即停止使用本程序。</p>\n' +
      '\n' +
      '<p>感谢您的理解与合作！</p>', '免责声明', {
        dangerouslyUseHTMLString: true,
        confirmButtonText: '我同意',
      })
})

type formData = {
  gradeTypeId: number | null,
  subjectId: number | null,
  gradeId: number | null,
  studyGroupId: number | null,
  entranceYearId: number | null,
  term: number | null,
}
const token = ref('')
const videosSelection = ref([])
const pptsSelection = ref([])
const videos = ref([])
const ppts = ref([])

const studyGroups = ref([])

// do not use same name with ref
const form: Ref<formData> = ref({
  gradeTypeId: null,
  subjectId: null,
  gradeId: null,
  studyGroupId: null,
  entranceYearId: null,
  term: null,
})

const previewData = ref([])
const options = ref([])
const terms = ref([])

const props: CascaderProps = {
  lazy: true,
  async lazyLoad(node, resolve) {
    const {level} = node
    let nodes
    if (level == 0) {
      nodes = (await loadGradeType()).map(item => ({
        value: item.gradeTypeId,
        label: item.gradeTypeName,
        leaf: false,
      }))
    }
    if (level == 1) {
      nodes = (await loadSubject(node.value)).map(item => ({
        value: item.subjectId,
        label: item.subjectName,
        leaf: false,
      }))
    }
    if (level == 2) {
      nodes = (await loadGrade(node.parent?.value, node.value)).map(item => ({
        value: item.gradeId,
        label: item.gradeName,
        leaf: true,
      }))
    }
    resolve(nodes)
  },
}

const studyGroupChange = async () => {
  form.value.entranceYearId = null
  form.value.term = null
  options.value = (await loadEntranceYears(previewData.value[0], previewData.value[1], previewData.value[2], form.value.studyGroupId)).map(item => ({
    value: item.entranceYearId,
    label: item.entranceYear
  }))
}

const entranceYearsChange = async () => {
  form.value.term = null
  terms.value = (await loadTerm(previewData[0], previewData[1], previewData[2], form.value.studyGroupId, form.value.entranceYearId))
}

const handleChange = async (previewData) => {
  form.value.studyGroupId = null
  form.value.entranceYearId = null
  form.value.term = null
  studyGroups.value = await loadStudyGroup(previewData[0], previewData[1], previewData[2])
}

const onSubmit = async () => {
  form.value.gradeTypeId = previewData.value[0]
  form.value.subjectId = previewData.value[1]
  form.value.gradeId = previewData.value[2]
  const loading = ElLoading.service({
    lock: true,
    text: '正在查找相关资源',
    background: 'rgba(0, 0, 0, 0.7)',
  })
  try {
    await collectResources(form.value)
  } catch (e) {
    loading.close()
    ElNotification({
      title: '错误',
      message: '参数错误或无数据',
      type: 'error',
    })
    return
  }
  loading.close()
}
const loadGradeType = async () => {
  const result = await axios.post('http://www.cdssxy.com/cloud-wlxy-customer/basedata/getGradeType.action')
  return result.data.data
}

const loadSubject = async (gradeTypeId: number) => {
  const result = await axios.post('http://www.cdssxy.com/cloud-wlxy-customer/basedata/getSubject.action', {
    gradeTypeId: gradeTypeId
  })
  return result.data.data
}

const loadGrade = async (gradeTypeId: number, subjectId: number) => {
  const result = await axios.post('http://www.cdssxy.com/cloud-wlxy-customer/basedata/getGrade.action', {
    gradeTypeId: gradeTypeId,
    subjectId: subjectId
  })
  return result.data.data
}
const loadStudyGroup = async (gradeTypeId: number, subjectId: number, gradeId: number) => {
  const result = await axios.post('http://www.cdssxy.com/cloud-wlxy-customer/basedata/getStudyGroup.action', {
    gradeTypeId: gradeTypeId,
    subjectId: subjectId,
    gradeId: gradeId
  })
  return result.data.data
}

const loadEntranceYears = async (gradeTypeId: number, subjectId: number, gradeId: number, studyGroupId: number) => {
  const result = await axios.post('http://www.cdssxy.com/cloud-wlxy-customer/basedata/getEntranceYears.action', {
    gradeTypeId: gradeTypeId,
    subjectId: subjectId,
    gradeId: gradeId,
    studyGroupId: studyGroupId
  },{
    headers:{
      "Authorization":token.value
    }
  })
  return result.data.data
}

const loadTerm = async (gradeTypeId: number, subjectId: number, gradeId: number, studyGroupId: number, entranceYearId: number) => {
  const result = await axios.post('http://www.cdssxy.com/cloud-wlxy-customer/basedata/getTerm.action', {
    gradeTypeId: gradeTypeId,
    subjectId: subjectId,
    gradeId: gradeId,
    studyGroupId: studyGroupId,
    entranceYearId: entranceYearId
  })
  return result.data.data
}

const getDownloadUrl = async (urlKey: string) => {
  const result = await axios.post('http://www.cdssxy.com/cloud-wlxy-customer/ossts/getPrivateDownloadUrl.action', {
    url: urlKey
  })
  return result.data.data
}

const getPPTs = async (pptId: number) => {
  const res = await axios.post('http://www.cdssxy.com/cloud-wlxy-customer/resourcedocument/findAllByFkCourseAndFkSysDirtreeId.action', {
    fkSysDirtreeId: pptId
  })
  return res.data.data
}

const getVideos = async (form: formData) => {
  const result = await axios.post('http://www.cdssxy.com/cloud-wlxy-customer/courseResource/videos.action', form)
  return result.data.data
}

const collectResources = async (form: formData) => {
  const res = await getVideos(form)
  videos.value = []
  ppts.value = []
  videosSelection.value = []
  pptsSelection.value = []
  const groupName = res[0].dirname
  let count = 0
  for (let i = 0; i < res[0].childNode.length; i++) {
    for (let j = 0; j < res[0].childNode[i].currentNodeResources.length; j++) {
      const url = res[0].childNode[i].currentNodeResources[j].browserUrl
      const name = res[0].childNode[i].currentNodeResources[j].resourceName + '.mp4'
      const downloadUrl = await getDownloadUrl(url)
      const video = {
        groupName: groupName,
        id: ++count,
        name: name, // 指定下载文件名（含扩展名）。【若不填此项，将根据下载 URL 自动获取文件名】
        url: downloadUrl, // 指定下载地址【必填项】
        dir: '' //指定文件的下载目录,相对于当前的downloadDir目录【一般不必填写，除非某些文件的下载地址的路径不符合你的需求】
      }
      videos.value.push(video)
    }
  }
  const pptId = res[0].sysDirtreeId
  const pptRes = await getPPTs(pptId)
  count = 0
  for (let i = 0; i < pptRes.length; i++) {
    const downloadUrl = (await axios.post('http://www.cdssxy.com/cloud-wlxy-customer/ossts/getPrivateDownloadUrl.action', {url: pptRes[i].browserUrl})).data.data
    const ppt = {
      groupName: groupName,
      id: ++count,
      name: pptRes[i].resourcename + '.' + pptRes[i].browserUrl.split(".")[pptRes[i].browserUrl.split(".").length - 1],
      url: downloadUrl,// 指定下载地址【必填项】
      dir: '' //指定文件的下载目录,相对于当前的downloadDir目录【一般不必填写，除非某些文件的下载地址的路径不符合你的需求】
    }
    ppts.value.push(ppt)
  }
  if (ppts.value.length == 0 || videos.value.length == 0) {
    throw new Error()
  }
}
const videosSelectionChange = (val) => {
  videosSelection.value = val
}
const pptsSelectionChange = (val) => {
  pptsSelection.value = val
}

const downloadVideos = async () => {
  await thunderDownload(videosSelection.value, videosSelection.value[0].groupName + '视频资源')
}

const downloadPPTs = async () => {
  await thunderDownload(pptsSelection.value, pptsSelection.value[0].groupName + '文档资源')

}

const thunderDownload = async (tasks, groupName: string) => {
  const loading = ElLoading.service({
    lock: true,
    text: '正在创建下载连接',
    background: 'rgba(0, 0, 0, 0.7)',
  })
  try {
    await thunderLink.newTask({
      downloadDir: 'cdssxy', // 指定批量任务的下载目录名称，迅雷会在用户剩余空间最大的磁盘根目录中创建这个目录。【若不填此项，会下载到用户默认下载目录】
      taskGroupName: groupName, // 指定任务组名称，可将批量任务合并成类似BT任务的【任务组】，迅雷将在下载目录中创建同名子文件夹保存所有下载文件。【推荐填写。若不填此项，迅雷下载列表会显示所有本次创建的下载任务，可能会使用户的下载列表显得杂乱】
      excludePath: '', // 如果您希望批量下载的文件在用户本地保持与服务器上相同的文件目录结构，可以指定排除URL的前缀，迅雷会根据被排除前缀后的URL路径，创建文件夹保存对应的文件。【本项需配合taskGroupName使用，通常用于下载绿色版游戏或程序。若不填此项，将把所有文件都放置于同一层下载目录中】
      installFile: '', // 指定批量任务中的安装程序或主程序，该任务组下载完成后，用户在迅雷中双击此任务组，将运行指定的文件。该任务组的图标也将变成指定文件的图标。【本项需配合taskGroupName使用，通常用于下载绿色版游戏或程序。若不填此项，下载完成后，用户双击此任务，将打开下载文件所在的文件夹】
      runParams: '',  // 指定打开安装程序或主程序时的启动参数【视需求填写】
      createShortcut: { // 下载完成后创建桌面快捷方式。【本项需配合taskGroupName使用，通常用于下载绿色版游戏或程序。若不填此项，将不在桌面创建快捷方式】
        name: '', // 快捷方式的名称【必填】
        targetFile: '', // 快捷方式指向的任务组内的文件相对路径【必填】
        runParams: '', // 运行参数【选填】
        startIn: '', // 起始位置[选填]
      },
      threadCount: '5', // 指定原始地址线程数【一般不必填写，但某些下载地址的服务器会限制单个IP的最大同时连接数，例如部分“网盘、在线视频”网站等，此时可将此项数值设为1，从而避免被服务器断开连接】
      hideYunPan: '1', // 是否隐藏下载到云盘入口
      referer: '', // 指定迅雷发起下载请求时上报的引用页【一般不必填写，除非某些下载地址的服务器会判断引用页】
      userAgent: 'Mozilla/5.0 (iPhone; CPU iPhone OS 13_2_3 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/13.0.3 Mobile/15E148 Safari/604.1 Edg/114.0.0.0', // 指定迅雷发起下载请求时上报的UA【一般不必填写，除非某些下载地址的服务器会判断UA】
      tasks: tasks
    })
  } catch (e) {
    loading.close()
    return
  }
  loading.close()
}
</script>

