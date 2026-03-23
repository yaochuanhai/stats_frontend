<template>
  <div class="container">
    <div class="header">
      <h1>📊 访问统计看板</h1>
      <p>实时查看系统使用情况</p>
    </div>
    
    <div v-if="loading" class="loading">
      <div class="loading-spinner">🔄</div>
      <p>正在加载数据...</p>
    </div>
    
    <div v-if="error" class="error">
      <span class="error-icon">⚠️</span>
      {{ error }}
    </div>
    
    <div v-if="!loading && !error && stats">
      <div class="date-info">
        <span class="date-badge">📅</span>
        统计日期：<strong>{{ stats.date }}</strong>
        <span class="last-update">（最后更新：{{ currentTime }}）</span>
      </div>
      
      <!-- 核心指标卡片 -->
      <div class="stats-grid">
        <div class="stat-card primary">
          <div class="stat-icon">👥</div>
          <div class="label">今日访问用户数</div>
          <div class="value">{{ stats.todayUsers }}</div>
          <div class="sub-label">独立访客（UV）</div>
        </div>
        <div class="stat-card secondary">
          <div class="stat-icon">📈</div>
          <div class="label">今日总访问次数</div>
          <div class="value">{{ stats.todayTotal }}</div>
          <div class="sub-label">访问总量（PV）</div>
        </div>
      </div>
      
      <!-- 页面统计表格 -->
      <div class="page-stats">
        <div class="stats-header">
          <h2>📈 各导航使用统计</h2>
          <div class="stats-summary">
            <span class="summary-item">
              <span class="summary-label">共</span>
              <span class="summary-value">{{ pageStats.length }}</span>
              <span class="summary-label">个页面</span>
            </span>
          </div>
        </div>
        
        <div v-if="!pageStats || pageStats.length === 0" class="empty-state">
          <span class="empty-icon">📭</span>
          <p>暂无数据</p>
        </div>
        
        <div v-else class="stats-table">
          <div class="table-header">
            <div class="col-page">页面名称</div>
            <div class="col-count">
              <span>🔥</span> 访问次数
            </div>
            <div class="col-users">
              <span>👤</span> 独立用户
            </div>
            <div class="col-percent">占比</div>
          </div>
          
          <div 
            v-for="(item, index) in sortedPageStats" 
            :key="index" 
            class="table-row"
            :class="{ 'top-row': index < 3 }"
          >
            <div class="col-page">
              <span class="rank-badge" v-if="index < 3">{{ index + 1 }}</span>
              <span class="page-name">{{ item.pageName }}</span>
            </div>
            <div class="col-count">
              <span class="count-value">{{ item.count }}</span>
            </div>
            <div class="col-users">
              <span class="users-value">{{ item.users }}</span>
            </div>
            <div class="col-percent">
              <div class="percent-bar">
                <div class="bar-fill" :style="{ width: item.percent + '%' }"></div>
              </div>
              <span class="percent-text">{{ item.percent }}%</span>
            </div>
          </div>
        </div>
      </div>
      
      <!-- IP 访问列表 -->
      <div class="ip-stats">
        <div class="stats-header">
          <h2>🌐 访问 IP 列表</h2>
          <div class="stats-summary">
            <span class="summary-item">
              <span class="summary-label">共</span>
              <span class="summary-value">{{ ipList.length }}</span>
              <span class="summary-label">个 IP</span>
            </span>
          </div>
        </div>
        
        <div v-if="!ipList || ipList.length === 0" class="empty-state">
          <span class="empty-icon">📭</span>
          <p>暂无 IP 数据</p>
        </div>
        
        <div v-else class="stats-table">
          <div class="table-header">
            <div class="col-rank">排名</div>
            <div class="col-ip">IP 地址</div>
            <div class="col-location">归属地</div>
            <div class="col-count">
              <span>🔥</span> 访问次数
            </div>
          </div>
          
          <div 
            v-for="(item, index) in sortedIpList" 
            :key="index" 
            class="table-row"
            :class="{ 'top-row': index < 3 }"
          >
            <div class="col-rank">
              <span class="rank-badge" v-if="index < 3">{{ index + 1 }}</span>
              <span v-else class="rank-normal">{{ index + 1 }}</span>
            </div>
            <div class="col-ip">
              <span class="ip-badge">{{ item.ip }}</span>
            </div>
            <div class="col-location">
              <span class="location-tag">📍 {{ item.location || '未知' }}</span>
            </div>
            <div class="col-count">
              <span class="count-value">{{ item.count }}</span>
            </div>
          </div>
        </div>
      </div>
      
      <button class="refresh-btn" @click="loadStats">
        <span class="btn-icon">🔄</span> 刷新数据
      </button>
    </div>
  </div>
</template>

<script>
import axios from 'axios'

// 创建 axios 实例
const api = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL
})

export default {
  name: 'App',
  data() {
    return {
      loading: true,
      error: null,
      stats: null,
      pageStats: [],
      ipList: [],
      currentTime: '',
      refreshInterval: null
    }
  },
  computed: {
    avgPerUser() {
      if (!this.stats?.todayUsers || this.stats.todayUsers === 0) {
        return '0'
      }
      const avg = (this.stats.todayTotal / this.stats.todayUsers).toFixed(1)
      return avg
    },
    sortedPageStats() {
      const total = this.pageStats.reduce((sum, item) => sum + item.count, 0)
      return this.pageStats
        .map(item => ({
          ...item,
          percent: total > 0 ? ((item.count / total) * 100).toFixed(1) : 0
        }))
        .sort((a, b) => b.count - a.count)
    },
    sortedIpList() {
      return [...this.ipList].sort((a, b) => b.count - a.count)
    }
  },
  mounted() {
    this.updateCurrentTime()
    this.loadStats()
    this.refreshInterval = setInterval(() => {
      this.loadStats()
      this.updateCurrentTime()
    }, 30000)
  },
  beforeUnmount() {
    if (this.refreshInterval) {
      clearInterval(this.refreshInterval)
    }
  },
  methods: {
    updateCurrentTime() {
      const now = new Date()
      this.currentTime = now.toLocaleTimeString('zh-CN', { 
        hour: '2-digit', 
        minute: '2-digit',
        second: '2-digit'
      })
    },
    async loadStats() {
      this.loading = true
      this.error = null
      try {
        const response = await api.get('/api/stats/today')
        this.stats = response.data
        
        if (response.data.pageCounts && response.data.pageUsers) {
          this.pageStats = Object.keys(response.data.pageCounts).map(pageName => ({
            pageName: pageName,
            count: response.data.pageCounts[pageName],
            users: response.data.pageUsers[pageName] || 0
          }))
        }
        
        // 处理 IP 数据
        if (response.data.ipList) {
          this.ipList = response.data.ipList.map(item => ({
            ip: item.ip,
            location: item.location,
            count: item.count
          }))
        }
        
        this.updateCurrentTime()
      } catch (err) {
        this.error = '加载数据失败：' + (err.response?.data?.message || err.message)
        console.error('加载统计数据失败:', err)
      } finally {
        this.loading = false
      }
    }
  }
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  min-height: 100vh;
  padding: 20px;
}

.container {
  max-width: 1400px;
  margin: 0 auto;
}

.header {
  text-align: center;
  color: white;
  margin-bottom: 40px;
}

.header h1 {
  font-size: 2.8em;
  margin-bottom: 10px;
  text-shadow: 2px 2px 4px rgba(0,0,0,0.2);
  font-weight: 700;
}

.header p {
  font-size: 1.2em;
  opacity: 0.95;
}

.loading {
  text-align: center;
  padding: 80px 20px;
  color: white;
}

.loading-spinner {
  font-size: 48px;
  animation: spin 1s linear infinite;
  margin-bottom: 20px;
}

@keyframes spin {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

.error {
  background: rgba(255, 107, 107, 0.95);
  color: white;
  padding: 20px;
  border-radius: 12px;
  margin-bottom: 20px;
  text-align: center;
  font-size: 16px;
  backdrop-filter: blur(10px);
}

.error-icon {
  font-size: 24px;
  margin-right: 10px;
}

.date-info {
  text-align: center;
  color: white;
  margin-bottom: 30px;
  font-size: 18px;
  background: rgba(255, 255, 255, 0.15);
  padding: 15px 25px;
  border-radius: 12px;
  backdrop-filter: blur(10px);
}

.date-badge {
  font-size: 22px;
  margin-right: 8px;
}

.last-update {
  opacity: 0.85;
  font-size: 14px;
  margin-left: 15px;
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 25px;
  margin-bottom: 35px;
}

.stat-card {
  background: white;
  border-radius: 16px;
  padding: 30px;
  box-shadow: 0 8px 16px rgba(0,0,0,0.12);
  transition: all 0.3s ease;
  position: relative;
  overflow: hidden;
}

.stat-card::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 4px;
}

.stat-card.primary::before {
  background: linear-gradient(90deg, #667eea, #764ba2);
}

.stat-card.secondary::before {
  background: linear-gradient(90deg, #f093fb, #f5576c);
}

.stat-card.tertiary::before {
  background: linear-gradient(90deg, #4facfe, #00f2fe);
}

.stat-card:hover {
  transform: translateY(-8px);
  box-shadow: 0 12px 24px rgba(0,0,0,0.18);
}

.stat-icon {
  font-size: 42px;
  margin-bottom: 15px;
}

.stat-card .label {
  font-size: 15px;
  color: #666;
  margin-bottom: 15px;
  font-weight: 500;
}

.stat-card .value {
  font-size: 3em;
  font-weight: bold;
  background: linear-gradient(135deg, #667eea, #764ba2);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  margin-bottom: 8px;
}

.stat-card .sub-label {
  font-size: 13px;
  color: #999;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.page-stats, .ip-stats {
  background: white;
  border-radius: 16px;
  padding: 30px;
  box-shadow: 0 8px 16px rgba(0,0,0,0.12);
  margin-bottom: 25px;
}

.stats-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 25px;
  padding-bottom: 20px;
  border-bottom: 2px solid #f0f0f0;
}

.stats-header h2 {
  font-size: 1.6em;
  color: #333;
  font-weight: 700;
}

.stats-summary {
  background: linear-gradient(135deg, #667eea, #764ba2);
  color: white;
  padding: 10px 20px;
  border-radius: 20px;
  font-size: 14px;
}

.summary-value {
  font-weight: bold;
  font-size: 18px;
  margin: 0 5px;
}

.empty-state {
  text-align: center;
  padding: 60px 20px;
  color: #999;
}

.empty-icon {
  font-size: 64px;
  display: block;
  margin-bottom: 15px;
}

.stats-table {
  border-radius: 12px;
  overflow: hidden;
}

.table-header {
  display: grid;
  grid-template-columns: 80px 1.5fr 2fr 1fr;
  gap: 15px;
  padding: 18px 20px;
  background: linear-gradient(135deg, #667eea, #764ba2);
  color: white;
  font-weight: 600;
  font-size: 15px;
}

.table-row {
  display: grid;
  grid-template-columns: 80px 1.5fr 2fr 1fr;
  gap: 15px;
  padding: 18px 20px;
  align-items: center;
  background: #f8f9fa;
  margin-bottom: 8px;
  border-radius: 8px;
  transition: all 0.3s ease;
}

.table-row:hover {
  background: #e9ecef;
  transform: translateX(5px);
}

.table-row.top-row {
  background: linear-gradient(135deg, #fff5f5, #f0f7ff);
  border-left: 4px solid #667eea;
}

.col-rank {
  display: flex;
  align-items: center;
  justify-content: center;
}

.rank-badge {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 32px;
  height: 32px;
  background: linear-gradient(135deg, #667eea, #764ba2);
  color: white;
  border-radius: 50%;
  font-weight: bold;
  font-size: 15px;
  flex-shrink: 0;
}

.rank-normal {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 32px;
  height: 32px;
  background: #e0e0e0;
  color: #666;
  border-radius: 50%;
  font-weight: 600;
  font-size: 14px;
  flex-shrink: 0;
}

.col-ip {
  display: flex;
  align-items: center;
}

.ip-badge {
  display: inline-block;
  background: linear-gradient(135deg, #4facfe, #00f2fe);
  color: white;
  padding: 8px 16px;
  border-radius: 8px;
  font-weight: 600;
  font-size: 15px;
  font-family: 'Courier New', monospace;
  letter-spacing: 0.5px;
}

.col-location {
  display: flex;
  align-items: center;
}

.location-tag {
  display: inline-flex;
  align-items: center;
  background: #f0f0f0;
  padding: 8px 14px;
  border-radius: 8px;
  font-size: 14px;
  color: #333;
  font-weight: 500;
}

.col-count, .col-users {
  text-align: center;
}

.count-value {
  display: inline-block;
  background: linear-gradient(135deg, #667eea, #764ba2);
  color: white;
  padding: 6px 16px;
  border-radius: 20px;
  font-weight: bold;
  font-size: 16px;
  min-width: 60px;
}

.users-value {
  display: inline-block;
  background: linear-gradient(135deg, #f093fb, #f5576c);
  color: white;
  padding: 6px 16px;
  border-radius: 20px;
  font-weight: bold;
  font-size: 16px;
  min-width: 60px;
}

.col-percent {
  display: flex;
  align-items: center;
  gap: 12px;
}

.percent-bar {
  flex: 1;
  height: 10px;
  background: #e0e0e0;
  border-radius: 10px;
  overflow: hidden;
}

.bar-fill {
  height: 100%;
  background: linear-gradient(90deg, #667eea, #764ba2);
  border-radius: 10px;
  transition: width 0.5s ease;
}

.percent-text {
  font-weight: bold;
  color: #667eea;
  min-width: 50px;
  text-align: right;
}

.refresh-btn {
  background: white;
  color: #667eea;
  border: 2px solid #667eea;
  padding: 15px 40px;
  border-radius: 30px;
  font-size: 17px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  margin: 25px auto;
  display: block;
  box-shadow: 0 4px 8px rgba(0,0,0,0.1);
}

.refresh-btn:hover {
  background: linear-gradient(135deg, #667eea, #764ba2);
  color: white;
  box-shadow: 0 6px 12px rgba(102, 126, 234, 0.4);
  transform: translateY(-2px);
}

.btn-icon {
  display: inline-block;
  margin-right: 8px;
  transition: transform 0.3s ease;
}

.refresh-btn:hover .btn-icon {
  animation: spin 0.8s linear infinite;
}

@media (max-width: 768px) {
  .header h1 {
    font-size: 2em;
  }
  
  .stats-grid {
    grid-template-columns: 1fr;
  }
  
  .table-header {
    display: none;
  }
  
  .table-row {
    grid-template-columns: 1fr;
    gap: 10px;
    padding: 15px;
  }
  
  .col-count, .col-users, .col-percent {
    text-align: left;
  }
}
</style>
