<template>
    <div>
        <!--The template row-->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">Home</el-breadcrumb-item>
            <el-breadcrumb-item :to="{ path: '/goods'}">Product list</el-breadcrumb-item>
            <el-breadcrumb-item>Add Product</el-breadcrumb-item>
        </el-breadcrumb>
        <el-card class="box-card">
            <el-alert title="Please fill the product details below" :closable="false" center type="success" show-icon/>
            <el-steps :active="active-0" finish-status="success" align-center>
                <el-step title="Basic Information"/>
                <el-step title="Product Details"/>
                <el-step title="Product Info."/>
                <el-step title="Product Image"/>
                <el-step title="Product content"/>
                <el-step title="Done"/>
            </el-steps>
<!--            The tabs-->
            <el-form :model="addForm" :rules="addFormRules" label-position="top" ref="addFormRef" label-width="120px" class="demo-addForm">

                <el-tabs type="border-card" :before-leave="beforeLeaveTab" v-model="active" tab-position="left" @tab-click="tabClicked" >
<!--                     before-leave="beforeLeaveTab(activeName, oldActiveName)"-->
                    <el-tab-pane name="0" label="Basic Information" >
                        <el-form-item label="Product name" prop="goods_name"  >
                            <el-input v-model="addForm.goods_name"  />
                        </el-form-item>
                        <el-form-item label="Product price"  prop="goods_price" >
                            <el-input v-model.number="addForm.goods_price" />
                        </el-form-item>
                        <el-form-item label="Product weight" prop="goods_weight" >
                            <el-input v-model.number="addForm.goods_weight" />
                        </el-form-item>
                        <el-form-item label="Stock" prop="goods_number" >
                            <el-input v-model.number="addForm.goods_number" />
                        </el-form-item>
                        <el-form-item label="Product name">
                            <el-cascader @change="handleChange" clearable v-model="addForm.goods_cat" :options="cateList" :props="cascaderProps"/>
                        </el-form-item>
                    </el-tab-pane>
                    <el-tab-pane name="1" label="Product Details">
                        <el-form-item v-for="item in manyData" :key="item.attr_id" label="item.attr_name"   >
                            <el-checkbox-group v-model="item.attr_vals">
                                <el-checkbox v-for="(value, i) in item.attr_vals" :key="i"  :label="value" border/>
                            </el-checkbox-group>
                        </el-form-item>
                    </el-tab-pane>
                    <el-tab-pane name="2" label="Product Info.">
                        <el-form-item v-for="item in onlyData" :key="item.attr_id" label="item.attr_name">
                            <el-input v-model="item.attr_vals"/>
                        </el-form-item>
                    </el-tab-pane>
                    <el-tab-pane name="3" label="Product Image">

                        <el-upload :on-success="handleSuccess" :headers="headersAtt" :action="uploadUrl" :on-preview="handlePreview" :on-remove="handleRemove"  list-type="picture" >
                            <el-button size="small" type="primary">Click to upload</el-button>
                        </el-upload>

                    </el-tab-pane>
                    <el-tab-pane name="4" label="Product Description">
                        <quill-editor v-model="addForm.goods_introduce"/>
                        <el-button type="primary" class="btn" @click="addProduct">Add the product</el-button>
                    </el-tab-pane>
                </el-tabs>
            </el-form>
        </el-card>

        <el-dialog title="Show the image"  :visible.sync="previewVisible" >
             <img :src="previewPath" class="image" alt="The product image" width="100%"/>
        </el-dialog>
    </div>
</template>

<script>
    import _ from 'lodash'
    export default
    {
        name: "AddProduct",
        data()
        {return{
            active:'0',
            cateList:[],
            manyData:[],
            onlyData:[],
            previewVisible:false,
            previewPath:"",
            uploadUrl:"https://www.tangxiaoyang.vip:8888/api/v2/upload",
            headersAtt:
                {
                    Authorization:JSON.parse(sessionStorage.getItem('userInfo')).token

                },
            cascaderProps:
                {
                    label:"cat_name",
                    value:"cat_id",
                    children:"children",
                    expandTrigger:"hover",
                    checkStrictly:false,//having the check box
                },
            addForm:
                {
                    goods_name:'',
                    goods_number:null,
                    goods_price:null,
                    goods_weight:null,
                    goods_cat:[],
                    goods_introduce:'',
                    pics:[],
                    attrs:[],
                },
            addFormRules:
            {
                goods_name:[{ required: true, message: 'Please input the product name', trigger: 'blur' }],
                goods_price:[{ required: true, message: 'Please input the product price', trigger: 'blur' }],
                goods_number:[{ required: true, message: 'Please input the quantity of the product', trigger: 'blur' }],
                goods_weight:[{ required: true, message: 'Please input the product weight', trigger: 'blur' }],
            },

        }},

        created()
        {
            this.getCateList()

        },
        computed:
            {
                cateId()
                {
                    return this.addForm.goods_cat.length ===3 ? this.addForm.goods_cat[2]:null
                },
            },
        methods:
            {
                async getCateList()
                {
                   const { data: res } = await this.$http.get('categories')
                   if (res.meta.status !== 200)
                   {
                       return this.$message.error("You have an error to get Category List")
                   }
                   this.cateList=res.data
                },
                handleChange()
                {
                    if(this.addForm.goods_cat.length!==3 )
                    {
                        this.addForm.goods_cat=[]
                    }

                },
                beforeLeaveTab()
                {
                    if(this.addForm.goods_cat.length!==3 )
                    {
                        this.$message.error("Please select the third group of the list")
                        return false
                    }
                },
                async tabClicked()
                {
                    if(this.active==='1')
                    {
                        const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`,
                            {params:{sel:'many'}})
                        if (res.meta.status !== 200)
                        {
                            this.$message.error("Getting the elements has failed")
                            return false
                        }
                        res.data.forEach(item => {item.attr_vals=item.attr_vals ? item.attr_vals.split(',') : []})
                        this.$message.success("You got the elements")
                        this.manyData=res.data
                    }

                    else if(this.active==='2')
                    {
                        const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`,{params:{sel:'only'}})
                        if (res.meta.status !== 200)
                        {
                            this.$message.error("Getting the active elements has failed")
                            return false
                        }
                        res.data.forEach(item => {item.attr_vals=item.attr_vals ? item.attr_vals.split(',') : []})
                        this.$message.success("You got the active elements")
                        this.onlyData=res.data
                    }

                },
                handlePreview(file)
                {
                    this.previewPath=file.url
                    this.previewVisible=true
                },
                handleRemove(file)
                {
                    const path=file.response.data.tmp_path
                    const al =this.addForm.pics.findIndex(item => {item.pic===path})
                    this.addForm.pics.splice(al,1)
                },
                handleSuccess(response)
                {
                    this.addForm.pics.push({pic:response.data.tmp_path})
                },
                addProduct()
                {
                    this.$refs.addFormRef.validate( async valid=>
                    {
                        if(!valid){return false}

                        //first pase the good cat and then send the request
                        // this.addForm.goods_cat=this.addForm.goods_cat.join(',')

                        const form =_.cloneDeep(this.addForm)
                        form.goods_cat=form.goods_cat.join(',')

                        this.manyData.forEach(item =>form.attrs.push({attr_id:item.attr_id ,  attr_value:item.attr_vals.join('')}))
                        this.onlyData.forEach(item =>form.attrs.push({attr_id:item.attr_id ,  attr_value:item.attr_vals}))


                        const { data: res } = await this.$http.post(`goods`,form)
                        if (res.meta.status !== 201){this.$message.error("There an error to add the product"); return false}

                        this.$message.success("You have successfully added a new product to the stock")
                        await this.$router.push("/goods")

                    })
                }



            }

    }
</script>

<style scoped>
    .el-breadcrumb{font:20px bold;}
    .el-checkbox{margin:10px;}
    .btn{margin-top:20px;}

</style>
