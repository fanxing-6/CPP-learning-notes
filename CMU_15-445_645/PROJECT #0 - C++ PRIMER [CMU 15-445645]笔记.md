# PROJECT #0 - C++ PRIMER [[CMU 15-445/645](https://15445.courses.cs.cmu.edu/fall2021/)]笔记





![image-20220614170304909](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202206141703030.png)

这是数据库领域的一门课程, 由[卡内基梅隆大学](https://www.zhihu.com/search?q=卡内基梅隆大学&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A"385197515"})副教授Andy Pavlo授课, 目前在网上有授课视频资料、实验以及配套的在线测评环境 (限时开放至2021年12月31日)

环境: wsl2 + Clion

## [Project #0 - C++ Primer ](https://15445.courses.cs.cmu.edu/fall2021/project0/)



还是很简单的,主要目的是让学生熟悉 C++17 的基本语法

代码如下:

```cpp
//===----------------------------------------------------------------------===//
//
//                         BusTub
//
// p0_starter.h
//
// Identification: src/include/primer/p0_starter.h
//
// Copyright (c) 2015-2020, Carnegie Mellon University Database Group
//
//===----------------------------------------------------------------------===//

#pragma once

#include <memory>
#include <stdexcept>
#include <vector>
#include "common/exception.h"
#include "common/logger.h"

namespace bustub {

/**
 * The Matrix type defines a common
 * interface for matrix operations.
 */
template <typename T>
class Matrix {
 protected:
  /**
   * TODO(P0): Add implementation
   *
   * Construct a new Matrix instance.
   * @param rows The number of rows
   * @param cols The number of columns
   *
   */
  Matrix(int rows, int cols) : rows_(rows), cols_(cols) {
    linear_ = new T[rows_ * cols_];
    memset(linear_, 0, rows_ * cols_);
  }

  /** The number of rows in the matrix */
  int rows_;
  /** The number of columns in the matrix */
  int cols_;

  /**
   * TODO(P0): Allocate the array in the constructor.
   * TODO(P0): Deallocate the array in the destructor.
   * A flattened array containing the elements of the matrix.
   */
  T *linear_;

 public:
  /** @return The number of rows in the matrix */
  virtual int GetRowCount() const = 0;

  /** @return The number of columns in the matrix */
  virtual int GetColumnCount() const = 0;

  /**
   * Get the (i,j)th matrix element.
   *
   * Throw OUT_OF_RANGE if either index is out of range.
   *
   * @param i The row index
   * @param j The column index
   * @return The (i,j)th matrix element
   * @throws OUT_OF_RANGE if either index is out of range
   */
  virtual T GetElement(int i, int j) const = 0;

  /**
   * Set the (i,j)th matrix element.
   *
   * Throw OUT_OF_RANGE if either index is out of range.
   *
   * @param i The row index
   * @param j The column index
   * @param val The value to insert
   * @throws OUT_OF_RANGE if either index is out of range
   */
  virtual void SetElement(int i, int j, T val) = 0;

  /**
   * Fill the elements of the matrix from `source`.
   *
   * Throw OUT_OF_RANGE in the event that `source`
   * does not contain the required number of elements.
   *
   * @param source The source container
   * @throws OUT_OF_RANGE if `source` is incorrect size
   */
  virtual void FillFrom(const std::vector<T> &source) = 0;

  /**
   * Destroy a matrix instance.
   * TODO(P0): Add implementation
   */
  virtual ~Matrix() {
    if (linear_ == nullptr) {
      return;
    }
    delete[] linear_;
  }
};

/**
 * The RowMatrix type is a concrete matrix implementation.
 * It implements the interface defined by the Matrix type.
 */
template <typename T>
class RowMatrix : public Matrix<T> {
 public:
  /**
   * TODO(P0): Add implementation
   *
   * Construct a new RowMatrix instance.
   * @param rows The number of rows
   * @param cols The number of columns
   */

  RowMatrix(int rows, int cols) : Matrix<T>(rows, cols) {
    data_ = new T *[rows];
    for (int i = 0; i < rows; ++i) {
      data_[i] = Matrix<T>::linear_ + i * cols;
    }
  }

  /**
   * TODO(P0): Add implementation
   * @return The number of rows in the matrix
   */
  int GetRowCount() const override { return this->rows_; }

  /**
   * TODO(P0): Add implementation
   * @return The number of columns in the matrix
   */
  int GetColumnCount() const override { return this->cols_; }

  /**
   * TODO(P0): Add implementation
   *
   * Get the (i,j)th matrix element.
   *
   * Throw OUT_OF_RANGE if either index is out of range.
   *
   * @param i The row index
   * @param j The column index
   * @return The (i,j)th matrix element
   * @throws OUT_OF_RANGE if either index is out of range
   */
  T GetElement(int i, int j) const override {
    if (i < 0 || i >= this->GetRowCount()) throw Exception(ExceptionType::OUT_OF_RANGE, "index is out of range");

    if (j < 0 || j >= this->GetColumnCount()) throw Exception(ExceptionType::OUT_OF_RANGE, "index is out of range");

    return std::move(data_[i][j]);
  }

  /**
   * Set the (i,j)th matrix element.
   *
   * Throw OUT_OF_RANGE if either index is out of range.
   *
   * @param i The row index
   * @param j The column index
   * @param val The value to insert
   * @throws OUT_OF_RANGE if either index is out of range
   */
  void SetElement(int i, int j, T val) override {
    if (i < 0 || i >= Matrix<T>::rows_ || j < 0 || j >= Matrix<T>::cols_) {
      throw Exception(ExceptionType::OUT_OF_RANGE, "index is out of range");
    } else
      data_[i][j] = val;
  }

  /**
   * TODO(P0): Add implementation
   *
   * Fill the elements of the matrix from `source`.
   *
   * Throw OUT_OF_RANGE in the event that `source`
   * does not contain the required number of elements.
   *
   * @param source The source container
   * @throws OUT_OF_RANGE if `source` is incorrect size
   */
  void FillFrom(const std::vector<T> &source) override {
    int rowCount = this->GetRowCount();
    int colCount = this->GetColumnCount();
    if (source.size() != static_cast<std::size_t>(rowCount * colCount)) {
      throw Exception(ExceptionType::OUT_OF_RANGE, "index is out of range");
    }
    for (int i = 0; i < rowCount * colCount; ++i) {
      Matrix<T>::linear_[i] = source[i];
    }
  }

  /**
   * TODO(P0): Add implementation
   *
   * Destroy a RowMatrix instance.
   */
  ~RowMatrix() override {
    //    for (int i = 0; i < this->GetRowCount(); ++i) {
    //      delete[] data_[i];
    //    }
    delete[] data_;
  }

 private:
  /**
   * A 2D array containing the elements of the matrix in row-major format.
   *
   * TODO(P0):
   * - Allocate the array of row pointers in the constructor.
   * - Use these pointers to point to corresponding elements of the `linear` array.
   * - Don't forget to deallocate the array in the destructor.
   */
  T **data_;
};

/**
 * The RowMatrixOperations class defines operations
 * that may be performed on instances of `RowMatrix`.
 */
template <typename T>
class RowMatrixOperations {
 public:
  /**
   * Compute (`matrixA` + `matrixB`) and return the result.
   * Return `nullptr` if dimensions mismatch for input matrices.
   * @param matrixA Input matrix
   * @param matrixB Input matrix
   * @return The result of matrix addition
   */
  static std::unique_ptr<RowMatrix<T>> Add(const RowMatrix<T> *matrixA, const RowMatrix<T> *matrixB) {
    if (matrixA->GetRowCount() != matrixB->GetRowCount() || matrixA->GetColumnCount() != matrixB->GetColumnCount())
      return std::unique_ptr<RowMatrix<T>>(nullptr);

    int rowCount = matrixA->GetRowCount();
    int colCount = matrixB->GetColumnCount();

    std::unique_ptr<RowMatrix<T>> mat = std::make_unique<RowMatrix<T>>(rowCount, colCount);

    for (int i = 0; i < rowCount; ++i) {
      for (int j = 0; j < colCount; ++j) {
        T temp = matrixA->GetElement(i, j) + matrixB->GetElement(i, j);
        mat->SetElement(i, j, std::move(temp));
      }
    }
    return mat;
  }

  /**
   * Compute the matrix multiplication (`matrixA` * `matrixB` and return the result.
   * Return `nullptr` if dimensions mismatch for input matrices.
   * @param matrixA Input matrix
   * @param matrixB Input matrix
   * @return The result of matrix multiplication
   */
  static std::unique_ptr<RowMatrix<T>> Multiply(const RowMatrix<T> *matrixA, const RowMatrix<T> *matrixB) {
    // TODO(P0): Add implementation
    if (matrixA->GetColumnCount() != matrixB->GetRowCount()) {
      return std::unique_ptr<RowMatrix<T>>(nullptr);
    }

    int row = matrixA->GetRowCount();
    int col = matrixB->GetColumnCount();

    T sum;

    std::unique_ptr<RowMatrix<T>> mat = std::make_unique<RowMatrix<T>>(row, col);
    for (int i = 0; i < row; ++i) {
      for (int j = 0; j < col; ++j) {
        sum = 0;
        for (int k = 0; k < matrixA->GetColumnCount(); ++k) {
          sum += matrixA->GetElement(i, k) * matrixB->GetElement(k, j);
        }
        mat->SetElement(i, j, sum);
      }
    }
    return mat;
  }

  /**
   * Simplified General Matrix Multiply operation. Compute (`matrixA` * `matrixB` + `matrixC`).
   * Return `nullptr` if dimensions mismatch for input matrices.
   * @param matrixA Input matrix
   * @param matrixB Input matrix
   * @param matrixC Input matrix
   * @return The result of general matrix multiply
   */
  static std::unique_ptr<RowMatrix<T>> GEMM(const RowMatrix<T> *matrixA, const RowMatrix<T> *matrixB,
                                            const RowMatrix<T> *matrixC) {
    std::unique_ptr tempMat = std::move(Multiply(matrixA, matrixB));
    if (tempMat == nullptr) {
      return std::unique_ptr<RowMatrix<T>>(nullptr);
    }
    return Add(tempMat, matrixC);
  }
};
}  // namespace bustub

```

