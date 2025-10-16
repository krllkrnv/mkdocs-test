<template>
  <div class="terms-list">
    <div class="header">
      <h1>Словарь терминов</h1>
      <div class="actions">
        <button @click="$router.push('/terms/create')" class="btn btn-primary">
          Добавить термин
        </button>
        <button @click="$router.push('/graph')" class="btn btn-secondary">
          График связей
        </button>
      </div>
    </div>

    <div v-if="loading" class="loading">
      Загрузка терминов...
    </div>

    <div v-else-if="error" class="error">
      Ошибка: {{ error }}
    </div>

    <div v-else class="terms-grid">
      <div 
        v-for="term in terms" 
        :key="term.id" 
        class="term-card"
        @click="$router.push(`/terms/${term.id}/edit`)"
      >
        <h3>{{ term.term }}</h3>
        <p class="category">{{ term.category }}</p>
        <p class="definition">{{ term.definition }}</p>
        <div v-if="term.related_terms && term.related_terms.length" class="related">
          <strong>Связанные термины:</strong>
          <span v-for="(related, index) in term.related_terms" :key="index">
            {{ related }}{{ index < term.related_terms.length - 1 ? ', ' : '' }}
          </span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted } from 'vue'
import api from '../services/api.js'

export default {
  name: 'TermsList',
  setup() {
    const terms = ref([])
    const loading = ref(true)
    const error = ref(null)

    const loadTerms = async () => {
      try {
        loading.value = true
        error.value = null
        const data = await api.getTerms(1, 100) // Запрашиваем все термины
        terms.value = data.terms // Берем массив терминов из ответа
      } catch (err) {
        error.value = err.message
        console.error('Ошибка загрузки терминов:', err)
      } finally {
        loading.value = false
      }
    }

    onMounted(() => {
      loadTerms()
    })

    return {
      terms,
      loading,
      error
    }
  }
}
</script>

<style scoped>
.terms-list {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 30px;
  padding-bottom: 20px;
  border-bottom: 2px solid #e0e0e0;
}

.header h1 {
  margin: 0;
  color: #2c3e50;
}

.actions {
  display: flex;
  gap: 10px;
}

.btn {
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-size: 14px;
  transition: background-color 0.3s;
}

.btn-primary {
  background-color: #3498db;
  color: white;
}

.btn-primary:hover {
  background-color: #2980b9;
}

.btn-secondary {
  background-color: #95a5a6;
  color: white;
}

.btn-secondary:hover {
  background-color: #7f8c8d;
}

.loading, .error {
  text-align: center;
  padding: 40px;
  font-size: 18px;
}

.error {
  color: #e74c3c;
}

.terms-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(350px, 1fr));
  gap: 20px;
}

.term-card {
  background: white;
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 20px;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.term-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0,0,0,0.15);
  border-color: #3498db;
}

.term-card h3 {
  margin: 0 0 10px 0;
  color: #2c3e50;
  font-size: 18px;
}

.category {
  margin: 0 0 10px 0;
  color: #7f8c8d;
  font-size: 12px;
  text-transform: uppercase;
  font-weight: bold;
}

.definition {
  margin: 0 0 15px 0;
  color: #34495e;
  line-height: 1.5;
}

.related {
  font-size: 12px;
  color: #7f8c8d;
  line-height: 1.4;
}

.related strong {
  color: #2c3e50;
}
</style>
