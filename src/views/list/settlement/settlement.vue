<template>
  <div class="container">
    <Breadcrumb :items="['menu.list', 'menu.list.settlement']" />
    <a-card class="general-card" :title="$t('menu.list.settlement')">
      <a-row>
        <a-col :flex="1">
          <a-form
            :model="formModel"
            :label-col-props="{ span: 6 }"
            :wrapper-col-props="{ span: 18 }"
            label-align="left"
          >
            <a-row :gutter="16">
              <a-col :span="8">
                <a-form-item
                  field="number"
                  :label="$t('settlement.form.category')"
                >
                  <a-input
                    v-model="formModel.number"
                    :placeholder="$t('settlement.form.category.placeholder')"
                  />
                </a-form-item>
              </a-col>
              <a-col :span="8">
                <a-form-item field="name" :label="$t('settlement.form.name')">
                  <a-input
                    v-model="formModel.name"
                    :placeholder="$t('settlement.form.name.placeholder')"
                  />
                </a-form-item>
              </a-col>

              <!--              <a-col :span="8">
                <a-form-item
                  field="status"
                  :label="$t('searchTable.form.status')"
                >
                  <a-select
                    v-model="formModel.status"
                    :options="statusOptions"
                    :placeholder="$t('searchTable.form.selectDefault')"
                  />
                </a-form-item>
              </a-col>-->
            </a-row>
          </a-form>
        </a-col>
        <a-divider style="height: 84px" direction="vertical" />
        <a-col :flex="'86px'" style="text-align: right">
          <a-space direction="vertical" :size="18">
            <a-button type="primary" @click="search">
              <template #icon>
                <icon-search />
              </template>
              {{ $t('searchTable.form.search') }}
            </a-button>
            <a-button @click="reset">
              <template #icon>
                <icon-refresh />
              </template>
              {{ $t('searchTable.form.reset') }}
            </a-button>
          </a-space>
        </a-col>
      </a-row>
      <a-divider style="margin-top: 0" />
      <a-row style="margin-bottom: 16px">
        <a-col :span="12">
          <!--          <a-space>
            <a-button type="primary">
              <template #icon>
                <icon-plus />
              </template>
              {{ $t('searchTable.operation.create') }}
            </a-button>
            <a-upload action="/">
              <template #upload-button>
                <a-button>
                  {{ $t('searchTable.operation.import') }}
                </a-button>
              </template>
            </a-upload>
          </a-space>-->
        </a-col>
        <a-col
          :span="12"
          style="display: flex; align-items: center; justify-content: end"
        >
          <a-button>
            <template #icon>
              <icon-download />
            </template>
            {{ $t('searchTable.operation.download') }}
          </a-button>
          <a-tooltip :content="$t('searchTable.actions.refresh')">
            <div class="action-icon" @click="search"
              ><icon-refresh size="18"
            /></div>
          </a-tooltip>
          <a-dropdown @select="handleSelectDensity">
            <a-tooltip :content="$t('searchTable.actions.density')">
              <div class="action-icon"><icon-line-height size="18" /></div>
            </a-tooltip>
            <template #content>
              <a-doption
                v-for="item in densityList"
                :key="item.value"
                :value="item.value"
                :class="{ active: item.value === size }"
              >
                <span>{{ item.name }}</span>
              </a-doption>
            </template>
          </a-dropdown>
          <a-tooltip :content="$t('searchTable.actions.columnSetting')">
            <a-popover
              trigger="click"
              position="bl"
              @popup-visible-change="popupVisibleChange"
            >
              <div class="action-icon"><icon-settings size="18" /></div>
              <template #content>
                <div id="tableSetting">
                  <div
                    v-for="(item, index) in showColumns"
                    :key="item.dataIndex"
                    class="setting"
                  >
                    <div style="margin-right: 4px; cursor: move">
                      <icon-drag-arrow />
                    </div>
                    <div>
                      <a-checkbox
                        v-model="item.checked"
                        @change="
                          handleChange($event, item as TableColumnData, index)
                        "
                      >
                      </a-checkbox>
                    </div>
                    <div class="title">
                      {{ item.title === '#' ? '序列号' : item.title }}
                    </div>
                  </div>
                </div>
              </template>
            </a-popover>
          </a-tooltip>
        </a-col>
      </a-row>
      <a-table
        row-key="id"
        :loading="loading"
        :pagination="pagination"
        :columns="(cloneColumns as TableColumnData[])"
        :data="renderData"
        :bordered="false"
        :size="size"
        @page-change="onPageChange"
      >
        <template #index="{ rowIndex }">
          {{ rowIndex + 1 + (pagination.current - 1) * pagination.pageSize }}
        </template>
        <template #contentType="{ record }">
          <a-space>
            <a-avatar
              v-if="record.contentType === 'img'"
              :size="16"
              shape="square"
            >
              <img
                alt="avatar"
                src="//p3-armor.byteimg.com/tos-cn-i-49unhts6dw/581b17753093199839f2e327e726b157.svg~tplv-49unhts6dw-image.image"
              />
            </a-avatar>
            <a-avatar
              v-else-if="record.contentType === 'horizontalVideo'"
              :size="16"
              shape="square"
            >
              <img
                alt="avatar"
                src="//p3-armor.byteimg.com/tos-cn-i-49unhts6dw/77721e365eb2ab786c889682cbc721c1.svg~tplv-49unhts6dw-image.image"
              />
            </a-avatar>
            <a-avatar v-else :size="16" shape="square">
              <img
                alt="avatar"
                src="//p3-armor.byteimg.com/tos-cn-i-49unhts6dw/ea8b09190046da0ea7e070d83c5d1731.svg~tplv-49unhts6dw-image.image"
              />
            </a-avatar>
            {{ $t(`searchTable.form.contentType.${record.contentType}`) }}
          </a-space>
        </template>
        <template #filterType="{ record }">
          {{ $t(`searchTable.form.filterType.${record.filterType}`) }}
        </template>
        <!--        <template #status="{ record }">
          <span v-if="record.status === 'offline'" class="circle"></span>
          <span v-else class="circle pass"></span>
          {{ $t(`searchTable.form.status.${record.status}`) }}
        </template>-->
        <!--        <template #operations>
          <a-button v-permission="['admin']" type="text" size="small">
            {{ $t('searchTable.columns.operations.view') }}
          </a-button>
          <a-button v-permission="['admin']" type="text" size="small">
            {{ $t('searchTable.columns.operations.waste') }}
          </a-button>
        </template>-->
      </a-table>
    </a-card>
  </div>
</template>

<script lang="ts" setup>
  import { computed, ref, reactive, watch, nextTick } from 'vue';
  import { useI18n } from 'vue-i18n';
  import useLoading from '@/hooks/loading';
  import { queryPolicyList, PolicyRecord, PolicyParams } from '@/api/list';
  import { Pagination } from '@/types/global';
  // import type { SelectOptionData } from '@arco-design/web-vue/es/select/interface';
  import type { TableColumnData } from '@arco-design/web-vue/es/table/interface';
  import cloneDeep from 'lodash/cloneDeep';
  import Sortable from 'sortablejs';

  type SizeProps = 'mini' | 'small' | 'medium' | 'large';
  type Column = TableColumnData & { checked?: true };

  const generateFormModel = () => {
    return {
      number: '',
      name: '',
      contentType: '',
      filterType: '',
      createdTime: [],
      status: '',
    };
  };
  const { loading, setLoading } = useLoading(true);
  const { t } = useI18n();
  const renderData = ref<PolicyRecord[]>([]);
  const formModel = ref(generateFormModel());
  const cloneColumns = ref<Column[]>([]);
  const showColumns = ref<Column[]>([]);

  const size = ref<SizeProps>('medium');

  const basePagination: Pagination = {
    current: 1,
    pageSize: 20,
  };
  const pagination = reactive({
    ...basePagination,
  });
  const densityList = computed(() => [
    {
      name: t('searchTable.size.mini'),
      value: 'mini',
    },
    {
      name: t('searchTable.size.small'),
      value: 'small',
    },
    {
      name: t('searchTable.size.medium'),
      value: 'medium',
    },
    {
      name: t('searchTable.size.large'),
      value: 'large',
    },
  ]);
  const columns = computed<TableColumnData[]>(() => [
    {
      title: t('searchTable.columns.index'),
      dataIndex: 'index',
      slotName: 'index',
    },
    {
      title: t('settlement.columns.category'),
      dataIndex: 'category',
    },
    {
      title: t('settlement.columns.name'),
      dataIndex: 'productName',
    },
    {
      title: t('settlement.columns.price'),
      dataIndex: 'price',
    },
    {
      title: t('settlement.columns.saleCount'),
      dataIndex: 'saleCount',
    },
    {
      title: t('settlement.columns.returnCount'),
      dataIndex: 'returnCount',
    },
    {
      title: t('settlement.columns.payment'),
      dataIndex: 'money',
    },
    {
      title: t('settlement.columns.returnPayment'),
      dataIndex: 'count',
    },
    {
      title: t('searchTable.columns.createdTime'),
      dataIndex: 'createdTime',
    },
    /*    {
      title: t('searchTable.columns.status'),
      dataIndex: 'status',
      slotName: 'status',
    }, */
    /*     {
      title: t('searchTable.columns.operations'),
      dataIndex: 'operations',
      slotName: 'operations',
    }, */
  ]);
  /*  const contentTypeOptions = computed<SelectOptionData[]>(() => [
    {
      label: t('searchTable.form.contentType.img'),
      value: 'img',
    },
    {
      label: t('searchTable.form.contentType.horizontalVideo'),
      value: 'horizontalVideo',
    },
    {
      label: t('searchTable.form.contentType.verticalVideo'),
      value: 'verticalVideo',
    },
  ]);
  const filterTypeOptions = computed<SelectOptionData[]>(() => [
    {
      label: t('searchTable.form.filterType.artificial'),
      value: 'artificial',
    },
    {
      label: t('searchTable.form.filterType.rules'),
      value: 'rules',
    },
  ]); */
  /*  const statusOptions = computed<SelectOptionData[]>(() => [
    {
      label: t('searchTable.form.status.online'),
      value: 'online',
    },
    {
      label: t('searchTable.form.status.offline'),
      value: 'offline',
    },
  ]); */
  const fetchData = async (
    params: PolicyParams = { current: 1, pageSize: 20 }
  ) => {
    setLoading(true);
    try {
      const { data } = await queryPolicyList(params);
      renderData.value = data.list;
      pagination.current = params.current;
      pagination.total = data.total;
    } catch (err) {
      // you can report use errorHandler or other
    } finally {
      setLoading(false);
    }
  };

  const search = () => {
    fetchData({
      ...basePagination,
      ...formModel.value,
    } as unknown as PolicyParams);
  };
  const onPageChange = (current: number) => {
    fetchData({ ...basePagination, current });
  };

  fetchData();
  const reset = () => {
    formModel.value = generateFormModel();
  };

  const handleSelectDensity = (
    val: string | number | Record<string, any> | undefined,
    e: Event
  ) => {
    size.value = val as SizeProps;
  };

  const handleChange = (
    checked: boolean | (string | boolean | number)[],
    column: Column,
    index: number
  ) => {
    if (!checked) {
      cloneColumns.value = showColumns.value.filter(
        (item) => item.dataIndex !== column.dataIndex
      );
    } else {
      cloneColumns.value.splice(index, 0, column);
    }
  };

  const exchangeArray = <T extends Array<any>>(
    array: T,
    beforeIdx: number,
    newIdx: number,
    isDeep = false
  ): T => {
    const newArray = isDeep ? cloneDeep(array) : array;
    if (beforeIdx > -1 && newIdx > -1) {
      // 先替换后面的，然后拿到替换的结果替换前面的
      newArray.splice(
        beforeIdx,
        1,
        newArray.splice(newIdx, 1, newArray[beforeIdx]).pop()
      );
    }
    return newArray;
  };

  const popupVisibleChange = (val: boolean) => {
    if (val) {
      nextTick(() => {
        const el = document.getElementById('tableSetting') as HTMLElement;
        const sortable = new Sortable(el, {
          onEnd(e: any) {
            const { oldIndex, newIndex } = e;
            exchangeArray(cloneColumns.value, oldIndex, newIndex);
            exchangeArray(showColumns.value, oldIndex, newIndex);
          },
        });
      });
    }
  };

  watch(
    () => columns.value,
    (val) => {
      cloneColumns.value = cloneDeep(val);
      cloneColumns.value.forEach((item, index) => {
        item.checked = true;
      });
      showColumns.value = cloneDeep(cloneColumns.value);
    },
    { deep: true, immediate: true }
  );
</script>

<script lang="ts">
  export default {
    name: 'SearchTable',
  };
</script>

<style scoped lang="less">
  .container {
    padding: 0 20px 20px 20px;
  }
  :deep(.arco-table-th) {
    &:last-child {
      .arco-table-th-item-title {
        margin-left: 16px;
      }
    }
  }
  .action-icon {
    margin-left: 12px;
    cursor: pointer;
  }
  .active {
    color: #0960bd;
    background-color: #e3f4fc;
  }
  .setting {
    display: flex;
    align-items: center;
    width: 200px;
    .title {
      margin-left: 12px;
      cursor: pointer;
    }
  }
</style>
