<template>
  <div class="container">
    <div class="header">
      <h1>📊 访问统计</h1>
      <p>实时查看系统使用情况</p>
    </div>
    
    <div v-if="loading" class="loading">
      <p>正在加载数据...</p>
    </div>
    
    <div v-if="error" class="error">
      {{ error }}
    </div>
    
    <div v-if="!loading && !error && stats">
      <div class="date-info">
        📅 统计日期：{{ stats.date }}
      </div>
      
      <div class="stats-grid">
        <div class="stat-card">
          <div class="label">今日访问用户数</div>
          <div class="value">{{ stats.todayUsers }}</div>
        </div>
        <div class="stat-card">
          <div class="label">今日总访问次数</div>
          <div class="value">{{ stats.todayTotal }}</div>
        </div>
      </div>
      
      <div class="page-stats">
        <h2>📈 各导航使用统计</h2>
        <div v-if="!pageStats || pageStats.length === 0" class="empty-state">
          暂无数据
        </div>
        <div v-else>
          <div v-for="(item, index) in pageStats" :key="index" class="page-item">
            <div class="page-name">{{ item.pageName }}</div>
            <div class="page-counts">
              <div class="count-item">
                <div class="label">访问次数</div>
                <div class="value">{{ item.count }}</div>
              </div>
              <div class="count-item">
                <div class="label">独立用户</div>
                <div class="value">{{ item.users }}</div>
              </div>
            </div>
          </div>
        </div>
      </div>
      
      <button class="refresh-btn" @click="loadStats">🔄 刷新数据</button>
    </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'App',
  data() {
    return {
      loading: true,
      error: null,
      stats: null,
      pageStats: []
    }
  },
  mounted() {
    this.loadStats()
    // 每30秒自动刷新
    this.refreshInterval = setInterval(() => {
      this.loadStats()
    }, 30000)
  },
  beforeUnmount() {
    if (this.refreshInterval) {
      clearInterval(this.refreshInterval)
    }
  },
  methods: {
    async loadStats() {
      this.loading = true
      this.error = null
      try {
        const response = await axios.get('/api/stats/today')
        this.stats = response.data
        
        // 处理页面统计数据
        if (response.data.pageCounts && response.data.pageUsers) {
          this.pageStats = Object.keys(response.data.pageCounts).map(pageName => ({
            pageName: pageName,
            count: response.data.pageCounts[pageName],
            users: response.data.pageUsers[pageName] || 0
          }))
        }
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
  max-width: 1200px;
  margin: 0 auto;
}

.header {
  text-align: center;
  color: white;
  margin-bottom: 30px;
}

.header h1 {
  font-size: 2.5em;
  margin-bottom: 10px;
  text-shadow: 2px 2px 4px rgba(0,0,0,0.2);
}

.header p {
  font-size: 1.1em;
  opacity: 0.9;
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 20px;
  margin-bottom: 30px;
}

.stat-card {
  background: white;
  border-radius: 12px;
  padding: 25px;
  box-shadow: 0 4px 6px rgba(0,0,0,0.1);
  transition: transform 0.3s, box-shadow 0.3s;
}

.stat-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 8px 12px rgba(0,0,0,0.15);
}

.stat-card .label {
  font-size: 14px;
  color: #666;
  margin-bottom: 10px;
}

.stat-card .value {
  font-size: 2.5em;
  font-weight: bold;
  color: #667eea;
}

.page-stats {
  background: white;
  border-radius: 12px;
  padding: 25px;
  box-shadow: 0 4px 6px rgba(0,0,0,0.1);
  margin-bottom: 20px;
}

.page-stats h2 {
  font-size: 1.5em;
  margin-bottom: 20px;
  color: #333;
  border-bottom: 2px solid #667eea;
  padding-bottom: 10px;
}

.empty-state {
  text-align: center;
  padding: 20px;
  color: #999;
}

.page-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 15px;
  margin-bottom: 10px;
  background: #f8f9fa;
  border-radius: 8px;
  transition: background 0.3s;
}

.page-item:hover {
  background: #e9ecef;
}

.page-name {
  font-size: 16px;
  font-weight: 600;
  color: #333;
}

.page-counts {
  display: flex;
  gap: 20px;
}

.count-item {
  text-align: center;
}

.count-item .label {
  font-size: 12px;
  color: #666;
  margin-bottom: 5px;
}

.count-item .value {
  font-size: 20px;
  font-weight: bold;
  color: #667eea;
}

.loading {
  text-align: center;
  padding: 50px;
  color: white;
  font-size: 18px;
}

.error {
  background: #ff6b6b;
  color: white;
  padding: 15px;
  border-radius: 8px;
  margin-bottom: 20px;
  text-align: center;
}

.refresh-btn {
  background: white;
  color: #667eea;
  border: 2px solid #667eea;
  padding: 12px 30px;
  border-radius: 8px;
  font-size: 16px;
  cursor: pointer;
  transition: all 0.3s;
  margin: 20px auto;
  display: block;
}

.refresh-btn:hover {
  background: #667eea;
  color: white;
}

.date-info {
  text-align: center;
  color: white;
  margin-bottom: 20px;
  font-size: 16px;
}
</style>
