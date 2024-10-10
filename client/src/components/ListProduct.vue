<template>
  <div>
    <h2>ListProducts</h2>
    <button @click="getProductById">Get detail</button>
    <button @click="removeProductById">Delete</button>
    <button @click="createProduct">Create Product</button>
    <button @click="updateProductById">Update Product</button>
    <ul>
      <li v-for="product in products" :key="product.id">
        {{ product.name }} - {{ product.price }}
      </li>
    </ul>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";

const products = ref([]);

const getAllProduct = async () => {
  try {
    const response = await fetch("http://localhost:8080/products");
    const data = await response.json();
    products.value = data;
    console.log("Danh sách sản phẩm:", products.value);
  } catch (error) {
    console.error("Không thể tải sản phẩm:", error);
  }
};

const getProductById = async () => {
  const id = 4;
  try {
    const response = await fetch(`http://localhost:8080/products/${id}`);
    if (!response.ok) {
      console.log("Không tìm thấy bản ghi");
      return;
    }
    const data = await response.json();
    console.log("Chi tiết sản phẩm:", data);
  } catch (error) {
    console.error("Lỗi khi lấy thông tin sản phẩm:", error);
  }
};

const removeProductById = async () => {
  const id = 4;
  try {
    const response = await fetch(`http://localhost:8080/products/${id}`, {
      method: "DELETE",
    });
    if (!response.ok) {
      console.log("Không thể xóa bản ghi");
      return;
    }
    getAllProduct();
  } catch (error) {
    console.error("Lỗi khi xóa sản phẩm:", error);
  }
};

const createProduct = async () => {
  const product = {
    name: "Đào",
    image:
      "https://cdn.eva.vn/upload/2-2022/images/2022-05-03/dao-3-1651587864-791-width600height400.jpg",
    price: 15000,
    quantity: 15,
  };

  try {
    const response = await fetch("http://localhost:8080/products", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify(product),
    });

    if (!response.ok) {
      console.log("Không thể tạo sản phẩm mới");
      return;
    }

    const result = await response.json();
    console.log("Kết quả tạo sản phẩm từ server:", result);
    getAllProduct();
  } catch (error) {
    console.error("Lỗi khi tạo sản phẩm:", error);
  }
};

const updateProductById = async () => {
  const id = 4;
  const product = {
    name: "Đào",
    image:
      "https://cdn.eva.vn/upload/2-2022/images/2022-05-03/dao-3-1651587864-791-width600height400.jpg",
    price: 30000,
    quantity: 120,
  };

  try {
    const response = await fetch(`http://localhost:8080/products/${id}`, {
      method: "PUT",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify(product),
    });

    if (!response.ok) {
      console.log("Không thể cập nhật sản phẩm");
      return;
    }

    const result = await response.json();
    console.log("Kết quả cập nhật sản phẩm từ server:", result);
    getAllProduct();
  } catch (error) {
    console.error("Lỗi khi cập nhật sản phẩm:", error);
  }
};

onMounted(() => {
  getAllProduct();
});
</script>

<style scoped></style>
