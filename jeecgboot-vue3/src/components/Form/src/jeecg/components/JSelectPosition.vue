<!--职务选择组件-->
<template>
  <div class="JSelectPosition">
    <JSelectBiz @handleOpen="handleOpen" :loading="loadingEcho" v-bind="attrs" @change="(changeValue) => $emit('update:value', changeValue)"></JSelectBiz>
    <!-- update-begin--author:liaozhiyang---date:20240515---for：【QQYUN-9260】必填模式下会影响到弹窗内antd组件的样式 -->
    <a-form-item>
      <PositionSelectModal @register="regModal" @getSelectResult="setValue" v-bind="getBindValue"></PositionSelectModal>
    </a-form-item>
    <!-- update-end--author:liaozhiyang---date:20240515---for：【QQYUN-9260】必填模式下会影响到弹窗内antd组件的样式 -->
  </div>
</template>
<script lang="ts">
  import PositionSelectModal from './modal/PositionSelectModal.vue';
  import JSelectBiz from './base/JSelectBiz.vue';
  import { defineComponent, ref, reactive, watchEffect, watch, provide, computed, unref } from 'vue';
  import { useModal } from '/@/components/Modal';
  import { propTypes } from '/@/utils/propTypes';
  import { useRuleFormItem } from '/@/hooks/component/useFormItem';
  import { useAttrs } from '/@/hooks/core/useAttrs';
  import { SelectValue } from 'ant-design-vue/es/select';

  export default defineComponent({
    name: 'JSelectPosition',
    components: {
      PositionSelectModal,
      JSelectBiz,
    },
    inheritAttrs: false,
    props: {
      value: propTypes.oneOfType([propTypes.string, propTypes.array]),
      labelKey: {
        type: String,
        default: 'name',
      },
      rowKey: {
        type: String,
        default: 'id',
      },
      params: {
        type: Object,
        default: () => {},
      },
    },
    emits: ['options-change', 'change', 'update:value'],
    setup(props, { emit, refs }) {
      const emitData = ref<any[]>();
      //注册model
      const [regModal, { openModal }] = useModal();
      //表单值
      const [state] = useRuleFormItem(props, 'value', 'change', emitData);
      //下拉框选项值
      const selectOptions = ref<SelectValue>([]);
      //下拉框选中值
      let selectValues = reactive<any>({
        value: [],
        change: false,
      });
      // 是否正在加载回显数据
      const loadingEcho = ref<boolean>(false);
      //下发 selectOptions,xxxBiz组件接收
      provide('selectOptions', selectOptions);
      //下发 selectValues,xxxBiz组件接收
      provide('selectValues', selectValues);
      //下发 loadingEcho,xxxBiz组件接收
      provide('loadingEcho', loadingEcho);

      const tag = ref(false);
      const attrs = useAttrs();

      /**
       * 监听组件值
       */
      // update-begin--author:liaozhiyang---date:20250423---for：【pull/8014】插槽方式弹窗中取消该数据checkbox的选中状态，需要点击第二次才生效。
      watch(
        () => props.value,
        () => {
          if (props.value) {
            initValue();
          } else {
            // update-begin--author:liaozhiyang---date:20250604---for：【issues/8233】resetFields时无法重置
            if (selectValues.value?.length) {
              selectValues.value = [];
            }
            // update-end--author:liaozhiyang---date:20250604---for：【issues/8233】resetFields时无法重置
          }
        },
        { deep: true, immediate: true }
      );
      // update-end--author:liaozhiyang---date:20250423---for：【pull/8014】插槽方式弹窗中取消该数据checkbox的选中状态，需要点击第二次才生效。

      /**
       * 监听selectValues变化
       */
      watch(selectValues, () => {
        if (selectValues) {
          state.value = selectValues.value;
        }
      });

      /**
       * 打卡弹出框
       */
      function handleOpen() {
        tag.value = true;
        openModal(true, {
          isUpdate: false,
        });
      }

      /**
       * 将字符串值转化为数组
       */
      function initValue() {
        let value = props.value ? props.value : [];
        if (value && typeof value === 'string' && value != 'null' && value != 'undefined') {
          state.value = value.split(',');
          selectValues.value = value.split(',');
        }
      }

      /**
       * 设置下拉框的值
       */
      function setValue(options, values) {
        selectOptions.value = options;
        //emitData.value = values.join(",");
        state.value = values;
        selectValues.value = values;
        //update-begin-author:liusq date:20230517 for:选择职务组件v-model方式绑定值不生效
        emit('update:value', values.join(','));
        //update-begin-author:liusq date:20230517 for:选择职务组件v-model方式绑定值不生效

      }

      const getBindValue = Object.assign({}, unref(props), unref(attrs));
      return {
        state,
        getBindValue,
        attrs,
        selectOptions,
        selectValues,
        loadingEcho,
        tag,
        regModal,
        setValue,
        handleOpen,
      };
    },
  });
</script>
<style lang="less" scoped>
  // update-begin--author:liaozhiyang---date:20240515---for：【QQYUN-9260】必填模式下会影响到弹窗内antd组件的样式
  .JSelectPosition {
    > .ant-form-item {
      display: none;
    }
  }
  // update-end--author:liaozhiyang---date:20240515---for：【QQYUN-9260】必填模式下会影响到弹窗内antd组件的样式
  .j-select-row {
    @width: 82px;

    .left {
      width: calc(100% - @width - 8px);
    }

    .right {
      width: @width;
    }

    .full {
      width: 100%;
    }

    :deep(.ant-select-search__field) {
      display: none !important;
    }
  }
</style>
