<template>
  <div class="order-detail-wrapper">
    <div class="order-detail-container">
      <a-card v-if="orderDetail" class="order-detail-card">
      <!-- 订单头部 -->
      <div class="order-header">
        <h2>订单详情</h2>
        <a-tag :color="getStatusColor(orderDetail.status)" style="font-size: 16px; padding: 4px 12px;">
          {{ orderDetail.statusName }}
        </a-tag>
      </div>

      <!-- 订单信息 -->
      <a-descriptions title="订单信息" bordered :column="2">
        <a-descriptions-item label="订单号">{{ orderDetail.orderNo }}</a-descriptions-item>
        <a-descriptions-item label="订单状态">{{ orderDetail.statusName }}</a-descriptions-item>
        <a-descriptions-item label="下单时间">{{ formatDate(orderDetail.createTime) }}</a-descriptions-item>
        <a-descriptions-item label="支付时间">
          {{ orderDetail.payTime ? formatDate(orderDetail.payTime) : '未支付' }}
        </a-descriptions-item>
        <a-descriptions-item label="支付方式">
          {{ orderDetail.payTypeName || '未支付' }}
        </a-descriptions-item>
        <a-descriptions-item label="物流单号">
          {{ orderDetail.logisticsNo || '暂无' }}
        </a-descriptions-item>
      </a-descriptions>

      <!-- 收货地址 -->
      <a-descriptions title="收货地址" bordered :column="1" style="margin-top: 24px;" v-if="orderDetail.address">
        <a-descriptions-item label="收货人">{{ orderDetail.address.receiver }}</a-descriptions-item>
        <a-descriptions-item label="联系电话">{{ orderDetail.address.phone }}</a-descriptions-item>
        <a-descriptions-item label="收货地址">{{ orderDetail.address.fullAddress }}</a-descriptions-item>
      </a-descriptions>

      <!-- 商品明细 -->
      <div class="order-items" style="margin-top: 24px;">
        <h3>商品明细</h3>
        <a-table
          :dataSource="orderDetail.items"
          :columns="itemColumns"
          :pagination="false"
          row-key="id"
        >
          <template #bodyCell="{ column, record }">
            <template v-if="column.key === 'title'">
              <div>
                <div>{{ record.title }}</div>
                <div style="color: #999; font-size: 12px;" v-if="record.skuTitle">
                  {{ record.skuTitle }}
                </div>
              </div>
            </template>
            <template v-else-if="column.key === 'price'">
              <span style="color: #ff4d4f;">¥{{ record.price }}</span>
            </template>
            <template v-else-if="column.key === 'subtotal'">
              <span style="color: #ff4d4f; font-weight: bold;">¥{{ record.subtotal }}</span>
            </template>
          </template>
        </a-table>
      </div>

      <!-- 金额统计 -->
      <div class="order-summary">
        <div class="summary-item">
          <span class="label">商品总额:</span>
          <span class="value">¥{{ orderDetail.totalAmount }}</span>
        </div>
        <div class="summary-item total">
          <span class="label">实付金额:</span>
          <span class="value">¥{{ orderDetail.payAmount }}</span>
        </div>
      </div>

      <!-- 订单备注 -->
      <div class="order-remark" v-if="orderDetail.remark">
        <h3>订单备注</h3>
        <p>{{ orderDetail.remark }}</p>
      </div>

      <!-- 操作按钮 -->
      <div class="order-actions">
        <a-button @click="goBack">返回</a-button>
        <a-button
          v-if="orderDetail.status === 0"
          type="primary"
          @click="handlePayOrder"
          style="margin-left: auto;"
        >
          <i class="fas fa-credit-card"></i>
          立即支付
        </a-button>
        <a-button
          v-if="orderDetail.status === 0 || orderDetail.status === 1"
          danger
          @click="handleCancelOrder"
        >
          取消订单
        </a-button>
        <a-button
          v-if="orderDetail.status === 2"
          type="primary"
          @click="handleConfirmOrder"
        >
          确认收货
        </a-button>
      </div>
    </a-card>

    <a-spin v-else class="loading-spinner" size="large" />
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import { useRouter, useRoute } from 'vue-router';
import { Modal, message } from 'ant-design-vue';
import { getOrderDetail, cancelOrder, confirmOrder, payOrder } from '@/api/OrderApi';

const router = useRouter();
const route = useRoute();

// 数据
const orderDetail = ref(null);

// 表格列定义
const itemColumns = [
  {
    title: '商品名称',
    key: 'title',
    dataIndex: 'title'
  },
  {
    title: '单价',
    key: 'price',
    dataIndex: 'price',
    width: 120
  },
  {
    title: '数量',
    key: 'quantity',
    dataIndex: 'quantity',
    width: 100
  },
  {
    title: '小计',
    key: 'subtotal',
    dataIndex: 'subtotal',
    width: 120
  }
];

// 加载订单详情
const loadOrderDetail = () => {
  const orderId = route.params.id;
  getOrderDetail(orderId, {
    onSuccess: (res) => {
      orderDetail.value = res;
    },
    onError: () => {
      router.push('/orders');
    }
  });
};

// 支付订单
const handlePayOrder = () => {
  Modal.confirm({
    title: '选择支付方式',
    content: '请选择支付方式完成支付',
    okText: '支付宝支付',
    cancelText: '微信支付',
    onOk: () => {
      // 支付宝支付
      payOrder(orderDetail.value.id, 'ALI', {
        successMsg: '支付成功',
        onSuccess: () => {
          loadOrderDetail();
        },
        onError: (error) => {
          message.error('支付失败：' + error.message);
        }
      });
    },
    onCancel: () => {
      // 微信支付
      payOrder(orderDetail.value.id, 'WECHAT', {
        successMsg: '支付成功',
        onSuccess: () => {
          loadOrderDetail();
        },
        onError: (error) => {
          message.error('支付失败：' + error.message);
        }
      });
    }
  });
};

// 取消订单
const handleCancelOrder = () => {
  Modal.confirm({
    title: '确认取消',
    content: '确定要取消该订单吗？',
    onOk: () => {
      cancelOrder(orderDetail.value.id, {
        successMsg: '订单已取消',
        onSuccess: () => {
          loadOrderDetail();
        }
      });
    }
  });
};

// 确认收货
const handleConfirmOrder = () => {
  Modal.confirm({
    title: '确认收货',
    content: '确定已收到商品吗？',
    onOk: () => {
      confirmOrder(orderDetail.value.id, {
        successMsg: '已确认收货',
        onSuccess: () => {
          loadOrderDetail();
        }
      });
    }
  });
};

// 返回
const goBack = () => {
  router.back();
};

// 获取状态颜色
const getStatusColor = (status) => {
  const colorMap = {
    0: 'orange',
    1: 'blue',
    2: 'cyan',
    3: 'green',
    4: 'default'
  };
  return colorMap[status] || 'default';
};

// 格式化日期
const formatDate = (dateStr) => {
  if (!dateStr) return '';
  const date = new Date(dateStr);
  return date.toLocaleString('zh-CN');
};

// 页面加载
onMounted(() => {
  loadOrderDetail();
});
</script>

<style scoped>
.order-detail-wrapper {
  min-height: 100vh;
  background: #faf8f3; /* 宣纸白 */
}

.order-detail-container {
  padding: 32px 20px;
}

.order-detail-card {
  max-width: 1200px;
  margin: 0 auto;
  box-shadow: 0 4px 20px rgba(139, 69, 19, 0.12);
  border: 1px solid #e8e8e8;
}

.order-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 28px;
  padding-bottom: 20px;
  border-bottom: 2px solid #e8e8e8;
}

.order-header h2 {
  margin: 0;
  font-size: 28px;
  font-weight: 700;
  color: #2c2c2c;
  font-family: 'SimSun', '宋体', serif;
  letter-spacing: 3px;
}

.order-items h3,
.order-remark h3 {
  margin-bottom: 20px;
  font-size: 20px;
  font-weight: 600;
  color: #2c2c2c;
  letter-spacing: 1px;
}

:deep(.ant-descriptions-item-label) {
  font-weight: 600;
  color: #666;
  letter-spacing: 0.5px;
  background: rgba(139, 69, 19, 0.03);
}

:deep(.ant-descriptions-item-content) {
  color: #2c2c2c;
  font-weight: 500;
}

.order-summary {
  margin-top: 20px;
  padding: 20px;
  background: linear-gradient(135deg, rgba(139, 69, 19, 0.04) 0%, rgba(197, 165, 114, 0.06) 100%);
  border-radius: 8px;
  border: 1px solid #e8e8e8;
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  gap: 12px;
}

.summary-item {
  display: flex;
  gap: 20px;
  font-size: 15px;
}

.summary-item.total {
  font-size: 20px;
  font-weight: 700;
}

.summary-item .label {
  color: #666;
  font-weight: 500;
}

.summary-item .value {
  color: #8b4513;
  font-weight: 700;
}

.order-remark {
  margin-top: 24px;
  padding: 20px;
  background: rgba(139, 69, 19, 0.02);
  border-radius: 8px;
  border: 1px solid #e8e8e8;
}

.order-remark p {
  margin: 0;
  color: #666;
  line-height: 1.8;
}

.order-actions {
  margin-top: 28px;
  display: flex;
  justify-content: flex-end;
  gap: 12px;
}

.order-actions :deep(.ant-btn-primary) {
  background: linear-gradient(135deg, #8b4513 0%, #a0522d 100%);
  border: none;
  font-weight: 600;
  letter-spacing: 2px;

  &:hover {
    transform: translateY(-2px);
    box-shadow: 0 4px 12px rgba(139, 69, 19, 0.4);
  }

  i {
    margin-right: 6px;
  }
}

.loading-spinner {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 400px;
  background: #fff;
  border-radius: 8px;
}
</style>

