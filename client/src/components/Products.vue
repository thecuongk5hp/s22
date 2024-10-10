<template>
  <div class="container">
    <h1>Quản lý sản phẩm</h1>
    <button class="add-button" @click="toggleForm">Thêm mới sản phẩm</button>

    <div style="display: flex">
      <table class="product-table">
        <thead>
          <tr>
            <th>#</th>
            <th>Tên sản phẩm</th>
            <th>Hình ảnh</th>
            <th>Giá</th>
            <th>Số lượng (kg)</th>
            <th>Ngày thêm</th>
            <th>Chức năng</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(product, index) in products" :key="product.id">
            <td>{{ index + 1 }}</td>
            <td>{{ product.name }}</td>
            <td>
              <img
                :src="product.image"
                :alt="product.name"
                class="product-image"
              />
            </td>
            <td>{{ product.price }} đ</td>
            <td>{{ product.quantity }}</td>
            <td>{{ product.date }}</td>
            <td>
              <button
                class="action-button edit"
                @click="editProduct(product.id)"
              >
                Sửa
              </button>
              <button
                class="action-button delete"
                @click="deleteProduct(product.id)"
              >
                Xóa
              </button>
            </td>
          </tr>
        </tbody>
      </table>
      <div v-if="isFormVisible" class="product-form">
        <h3 style="margin: 0; font-size: 18px">
          {{ currentProductId ? "Cập nhật sản phẩm" : "Thêm mới sản phẩm" }}
        </h3>
        <form @submit.prevent="submitForm">
          <div class="form-group">
            <label for="name">Tên sản phẩm</label>
            <input v-model="newProduct.name" type="text" id="name" required />
          </div>
          <div class="form-group">
            <label for="price">Giá</label>
            <input
              v-model="newProduct.price"
              type="number"
              id="price"
              required
            />
          </div>
          <div class="form-group">
            <label for="quantity">Số lượng (kg)</label>
            <input
              v-model="newProduct.quantity"
              type="number"
              id="quantity"
              required
            />
          </div>
          <div class="form-group">
            <label for="image">Hình ảnh</label>
            <input v-model="newProduct.image" type="url" id="image" required />
          </div>
          <div class="form-buttons">
            <button type="submit" class="save-button">
              {{ currentProductId ? "Cập nhật" : "Lưu" }}
            </button>
            <button type="button" class="cancel-button" @click="toggleForm">
              Hủy
            </button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";

// Dữ liệu sản phẩm
const products = ref([]);

// Biến để quản lý hiển thị form
const isFormVisible = ref(false);

// Biến để lưu sản phẩm đang được sửa
const currentProductId = ref(null);

// Khởi tạo sản phẩm mới
const newProduct = ref({
  name: "",
  price: "",
  quantity: "",
  image: "",
});

// Fetch danh sách sản phẩm từ server
const fetchProducts = async () => {
  try {
    const response = await fetch("http://localhost:8080/products");
    const data = await response.json();
    products.value = data;
  } catch (error) {
    console.error("Error fetching products:", error);
  }
};

// Toggle trạng thái form hiển thị/ẩn
const toggleForm = () => {
  isFormVisible.value = !isFormVisible.value;
  if (!isFormVisible.value) {
    resetForm();
  }
};

// Reset form về trạng thái ban đầu
const resetForm = () => {
  newProduct.value = {
    name: "",
    price: "",
    quantity: "",
    image: "",
  };
};

const editProduct = (id) => {
  const product = products.value.find((p) => p.id === id);
  if (product) {
    newProduct.value = { ...product };
    currentProductId.value = id;
    isFormVisible.value = true;
  }
};

// Xử lý khi submit form
const submitForm = async () => {
  if (validateForm()) {
    try {
      if (currentProductId.value) {
        // Nếu đang sửa sản phẩm, thực hiện cập nhật
        const updatedProduct = {
          ...newProduct.value,
          date: new Date().toLocaleDateString("vi-VN"),
        };

        const response = await fetch(
          `http://localhost:8080/products/${currentProductId.value}`,
          {
            method: "PUT",
            headers: {
              "Content-Type": "application/json",
            },
            body: JSON.stringify(updatedProduct),
          }
        );

        if (response.ok) {
          // Cập nhật sản phẩm trong danh sách
          const index = products.value.findIndex(
            (p) => p.id === currentProductId.value
          );
          if (index !== -1) {
            products.value[index] = updatedProduct;
          }
          toggleForm();
        } else {
          console.error("Failed to update product");
        }
      } else {
        // Thêm sản phẩm mới
        const productToAdd = {
          ...newProduct.value,
          date: new Date().toLocaleDateString("vi-VN"),
        };

        const response = await fetch("http://localhost:8080/products", {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify(productToAdd),
        });

        if (response.ok) {
          products.value.push(productToAdd);
          toggleForm();
        } else {
          console.error("Failed to add product");
        }
      }
    } catch (error) {
      console.error("Error saving product:", error);
    }
  }
};

// Xác thực form
const validateForm = () => {
  if (
    !newProduct.value.name ||
    !newProduct.value.price ||
    !newProduct.value.quantity ||
    !newProduct.value.image
  ) {
    alert("Vui lòng điền đầy đủ thông tin.");
    return false;
  }

  // Kiểm tra trùng tên sản phẩm
  const isDuplicate = products.value.some(
    (product) => product.name === newProduct.value.name
  );
  if (isDuplicate) {
    alert("Tên sản phẩm đã tồn tại.");
    return false;
  }

  return true;
};

const deleteProduct = async (id) => {
  try {
    const response = await fetch(`http://localhost:8080/products/${id}`, {
      method: "DELETE",
    });

    if (response.ok) {
      // Xóa sản phẩm khỏi danh sách
      products.value = products.value.filter((p) => p.id !== id);
    } else {
      console.error("Failed to delete product");
    }
  } catch (error) {
    console.error("Error deleting product:", error);
  }
};

// Fetch sản phẩm khi component được mount
onMounted(() => {
  fetchProducts();
});
</script>

<style scoped>
/* Styles cho form và bảng */
.container {
  font-family: Arial, sans-serif;
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}
.add-button,
.save-button,
.cancel-button {
  background-color: #4285f4;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  margin-bottom: 20px;
}
.cancel-button {
  background-color: #ff5252;
  margin-left: 10px;
}
.product-form {
  padding: 20px;
  border-radius: 8px;
}
.form-group {
  display: flex;
  flex-direction: column;
  margin: 10px 0;
}
.product-table {
  width: 100%;
  border-collapse: collapse;
}
.product-table th,
.product-table td {
  border: 1px solid #e0e0e0;
  padding: 12px;
  text-align: center;
}
.product-image {
  width: 50px;
  height: 50px;
  object-fit: cover;
}
.action-button {
  padding: 6px 12px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 12px;
  margin-right: 5px;
}
.action-button.edit {
  background-color: #ffa000;
  color: white;
}
.action-button.delete {
  background-color: #ff5252;
  color: white;
}
</style>
