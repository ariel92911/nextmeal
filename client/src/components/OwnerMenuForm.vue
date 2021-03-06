<template>
  <form
    class="form p-4 rounded shadow-sm"
    novalidate
  >
    <h3 class="form-header mb-4">
      更新下週菜單
    </h3>
    <!--Dish selection-->
    <CustomSelect
      v-model="id"
      :options="options"
      :v="$v.id"
      :target="'id'"
    >
      <template v-slot:label>
        餐點
      </template>
      <template v-slot:option>
        選擇餐點
      </template>
      <template v-slot:invalid>
        請選擇一種餐點
      </template>
    </CustomSelect>

    <!--Quantity-->
    <div
      class="form-group"
      :class="{invalid: $v.quantity.$error}"
    >
      <label for="quantity">供應數量</label>
      <input
        id="quantity"
        v-model.number="quantity"
        type="number"
        class="form-control"
        min="1"
        max="50"
        required
        @input="$v.quantity.$touch()"
      >
      <small
        v-if="$v.quantity.$error"
        class="form-text"
      >請填寫欲提供的數量，最少 1 份和最多 50 份</small>
    </div>

    <div class="btn-container mt-4">
      <ProcessButton
        class="btn"
        :is-processing="isProcessing"
        :v="$v"
        @after-click="handleSubmit"
      >
        <template #initial>
          更新
        </template>
      </ProcessButton>
    </div>
  </form>
</template>

<script>
import CustomSelect from '../components/CustomSelect'
import ProcessButton from '../components/Button/ProcessButton'
import ownerAPI from '../apis/owner'
import { Toast } from '../utils/helpers'
import { required, between } from 'vuelidate/lib/validators'

export default {
  components: {
    CustomSelect,
    ProcessButton
  },
  props: {
    initialMeal: {
      type: Object,
      default: () => ({
        id: '',
        name: '',
        nextServing_quantity: 0
      })
    },
    options: {
      type: Array,
      required: true
    }
  },
  data () {
    return {
      quantity: 0,
      id: '',
      isProcessing: false
    }
  },
  validations: {
    id: {
      required
    },
    quantity: {
      required,
      between: between(1, 50)
    }
  },
  watch: {
    initialMeal (meal) {
      this.id = meal.id || this.id
      this.quantity = meal.nextServing_quantity || this.quantity
    }
  },
  created () {
    this.id = this.initialMeal.id || this.id
    this.quantity = this.initialMeal.nextServing_quantity || this.quantity
  },
  methods: {
    async handleSubmit (e) {
      // prepare form data
      const formData = {
        id: this.id,
        quantity: this.quantity
      }

      // update loading status
      this.isProcessing = true

      try {
        // send the updated data
        const { data, statusText } = await ownerAPI.menu.putMenu(formData)
        // error handling
        if (data.status !== 'success' || statusText !== 'OK') throw new Error(data.message)
        // send dish name and image from response to parent
        this.$emit('after-submit', { id: formData.id, nextServing_quantity: formData.quantity, name: data.meal.name, image: data.meal.image })
        // notify for successful update
        Toast.fire({
          type: 'success',
          title: '成功更新下週菜單！'
        })
        // update loading status
        this.isProcessing = false
      } catch (error) {
        // update loading status
        this.isProcessing = false
        // fire error messages
        Toast.fire({
          type: 'error',
          title: '無法更新下週菜單，請稍後再試'
        })
      }
    }
  }
}
</script>

<style lang="scss" scoped>
.form {
  @include inputValidation;
  @include formControl;
  color: color(secondary);
  background-color: color(quaternary);

  &-header { font-size: size(md); }
}

.btn-container { text-align: center; }
</style>
