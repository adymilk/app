<template>
    <div id="add-address">
        <group label-width="100px">
            <x-input title="姓名 : "
                is-type="china-name"
                required
                show-clear
                v-model="form.name"
                placeholder="请输入收货姓名"></x-input>
            <x-input title="电话 : "
                required
                is-type="china-mobile"
                show-clear
                v-model="form.tel"
                placeholder="请输入收货电话"
                type="tel"></x-input>
            <x-address title="收货地址 : "
                :list="addressData"
                v-model="address"
                placeholder="请选择收货地址"></x-address>
            <x-textarea :max="100"
                placeholder="请输入收货详细地址"
                v-model="form.address"></x-textarea>
        </group>
        <div class="submit"
            v-if="$route.query.id"
            @click="addAddress()">确认修改</div>
        <div class="submit"
            v-else
            @click="addAddress()">确认添加</div>
    </div>
</template>

<script>
import { Group, XInput, XTextarea, XAddress } from 'vux'
import { mapMutations } from 'vuex'
import address from '@/components/common/js/cityData'
import request from '@/components/common/js/request'
export default {
    name: 'add-address',
    created() {
        if (this.$route.query.id) {
            request({
                url: '/shop/address/getAddressDetail',
                method: 'get',
                params: { address_id: this.$route.query.id }
            }).then(res => {
                this.form = res.data
                this.form.address_id = this.$route.query.id
                this.address = [res.data.province_id + '', res.data.city_id + '', res.data.area_id + '']
            }).catch(err => {
                console.log(err)
            })
        }
    },
    data() {
        return {
            addressData: address,
            form: {
                address_id: '',
                name: '',
                tel: '',
                province: 0,
                area: 0,
                city: 0,
                address: '',
                loading: false,
            },
            address: [],
        }
    },
    methods: {
        addAddress() {
            if (this.form.name.length == 0 || this.form.name.length > 4) {
                this.$vux.toast.text('输入的姓名错误')
                return
            }
            if (this.form.tel.length != 11) {
                this.$vux.toast.text('输入的电话错误')
                return
            }
            if (this.form.province == 0 || this.form.area == 0 || this.form.city == 0) {
                console.log(this.form.province, this.form.area, this.form.city)
                this.$vux.toast.text('输入的地址有误,请选择省市区')
                return
            }
            if (this.form.address == '') {
                this.$vux.toast.text('输入的地址有误,请填写详细收货地址')
                return
            }
            if (this.loading) {
                return
            }
            this.loading = true
            request({
                url: '/shop/address/addAddress',
                method: 'post',
                data: this.form,
            }).then(res => {
                if (this.form.address_id > 0) {
                    this.editAddress({ address: res.data, index: this.$route.query.index })
                } else {
                    this.addNewAddress(res.data)
                }
                this.$router.back()
            }).catch(err => {
                console.log(err)
            })
        },
        ...mapMutations({ addNewAddress: 'ADD_NEW_ADDRESS', editAddress: 'EDIT_ADDRESS' })
    },
    components: {
        Group, XInput, XTextarea, XAddress
    },
    watch: {
        address(newVal) {
            [this.form.province, this.form.area, this.form.city] = [newVal[0], newVal[1], newVal[2]]
        }
    }
}
</script>

<style lang="less">
#add-address {
    .weui-cells__title {
        font-size: 17px;
        color: #000;
    }
    .weui-input::-webkit-input-placeholder {
        font-size: 13px;
        color: #ddd;
    }
    .weui-input::-moz-input-placeholder {
        font-size: 13px;
        color: #ddd;
    }
    .weui-input::-ms-input-placeholder {
        font-size: 13px;
        color: #ddd;
    }
    .weui-textarea::-webkit-input-placeholder {
        font-size: 13px;
        color: #ddd;
    }
    .weui-textarea::-moz-input-placeholder {
        font-size: 13px;
        color: #ddd;
    }
    .weui-textarea::-ms-input-placeholder {
        font-size: 13px;
        color: #ddd;
    }
    .vux-popup-picker-select {
        text-align: left !important;
        .vux-cell-placeholder {
            font-size: 13px;
            color: #ddd;
        }
    }
    .weui-cells:before {
        border-top: none;
    }
    .submit {
        position: absolute;
        left: 0;
        right: 0;
        bottom: 0;
        margin: 20px;
        padding: 10px;
        font-size: 15px;
        text-align: center;
        background-color: #dc7433;
        color: #fff;
        border-radius: 25px;
    }
}
</style>
