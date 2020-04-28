<template>
  <div class="w-75">
    <a-form :form="form.fc" v-bind="formItemLayout">
      <a-form-item label="类型">
        <a-radio-group v-model="form.fd.type">
          <a-radio-button value="keypair">镜像仓库密钥</a-radio-button>
          <a-radio-button value="cephCSI">Ceph CSI</a-radio-button>
        </a-radio-group>
      </a-form-item>
      <a-form-item label="名称">
        <a-input placeholder="请输入名称" v-decorator="decorators.name" />
      </a-form-item>
      <a-form-item label="集群">
        <cluster-select v-decorator="decorators.cluster" :clusterObj.sync="clusterObj" />
      </a-form-item>
      <a-form-item label="命名空间">
        <namespace-select v-decorator="decorators.namespace" :cluster="clusterObj.id" :namespaceObj.sync="namespaceObj" />
      </a-form-item>
      <template v-if="form.fd.type === 'keypair'">
        <a-form-item label="镜像仓库地址" prop="server">
          <a-input v-decorator="decorators.server" placeholder="请输入镜像仓库地址" />
        </a-form-item>
        <a-form-item label="用户名" prop="user">
          <a-input v-decorator="decorators.user" placeholder="请输入用户名" />
        </a-form-item>
        <a-form-item label="密码" prop="password">
          <a-input v-decorator="decorators.password" placeholder="请输入密码，" show-password />
        </a-form-item>
        <a-form-item label="邮箱" prop="email">
          <a-input v-decorator="decorators.email" placeholder="请输入邮箱" />
        </a-form-item>
      </template>
      <template v-else>
        <a-form-item label="UserId" prop="userId">
          <a-input v-decorator="decorators.userId" placeholder="例如 admin" />
        </a-form-item>
        <a-form-item label="UserKey" prop="userKey">
          <a-input v-decorator="decorators.userKey" placeholder="例如 AQAc8m5e754hHhAAvWYtFivfs9bvhRm6P51QXA==" />
        </a-form-item>
      </template>
    </a-form>
  </div>
</template>

<script>
import ClusterSelect from '@K8S/sections/ClusterSelect'
import NamespaceSelect from '@K8S/sections/NamespaceSelect'
import { SECRET_DEFAULT_TYPE } from '@K8S/constants'

export default {
  name: 'K8sConfigmapCreate',
  components: {
    ClusterSelect,
    NamespaceSelect,
  },
  data () {
    return {
      form: {
        fc: this.$form.createForm(this),
        fd: {
          type: 'keypair',
        },
      },
      formItemLayout: {
        labelCol: { span: 4 },
        wrapperCol: { span: 20 },
      },
      decorators: {
        name: [
          'name',
          {
            validateFirst: true,
            rules: [
              { required: true, message: '请输入名称' },
              { min: 2, max: 24, message: '长度在 2 到 24 个字符', trigger: 'blur' },
              { validator: this.$validate('resourceName') },
            ],
          },
        ],
        cluster: [
          'cluster',
          {
            rules: [
              { required: true, message: '请选择集群' },
            ],
          },
        ],
        namespace: [
          'namespace',
          {
            rules: [
              { required: true, message: '请选择命名空间' },
            ],
          },
        ],
        server: [
          'server',
          {
            rules: [
              { required: true, message: '请输入镜像仓库地址', trigger: 'blur' },
            ],
          },
        ],
        user: [
          'user',
          {
            rules: [
              { required: true, message: '请输入用户名', trigger: 'blur' },
            ],
          },
        ],
        password: [
          'password',
          {
            rules: [
              { required: true, message: '请输入密码', trigger: 'blur' },
            ],
          },
        ],
        userId: [
          'userId',
          {
            rules: [
              { required: true, message: '请输入UserId', trigger: 'blur' },
            ],
          },
        ],
        userKey: [
          'userKey',
          {
            rules: [
              { required: true, message: '请输入UserKey', trigger: 'blur' },
            ],
          },
        ],
        email: [
          'email',
          {
            validateFirst: true,
            rules: [
              { required: true, message: '请输入邮箱地址', trigger: 'blur' },
              { validator: this.$validate('email'), trigger: 'blur' },
            ],
          },
        ],

      },
      clusterObj: {},
      namespaceObj: {},
    }
  },
  methods: {
    async _doCreate (data, resouce) {
      await new this.$Manager('secrets', 'v1').create({ data })
    },
    async doCreate () {
      try {
        const values = await this.form.fc.validateFields({ scroll: { alignWithTop: true, offsetTop: 100 } })
        const params = {
          name: values.name,
          cluster: values.cluster,
          namespace: values.namespace,
          type: SECRET_DEFAULT_TYPE,
        }
        if (this.form.fd.type === 'keypair') {
          params.dockerConfigJson = {
            email: values.email,
            password: values.password,
            user: values.user,
            server: values.server,
          }
        } else {
          params.type = 'yunion.io/ceph-csi'
          params.cephCSI = {}
          params.cephCSI.userId = values.userId
          params.cephCSI.userKey = values.userKey
        }
        await this._doCreate(params)
        this.$message.success('操作成功')
      } catch (error) {
        throw error
      }
    },
  },
}
</script>