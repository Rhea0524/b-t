# Android Budget Tracker App - Complete Implementation Guide

This guide provides a step-by-step approach to implementing the budget tracker app as specified in the requirements. The app will be built using Android Studio, following MVVM architecture, and utilizing Room database for local data persistence.

## Table of Contents
1. [Project Setup](#project-setup)
2. [Database Implementation](#database-implementation)
3. [Repository Implementation](#repository-implementation)
4. [ViewModel Implementation](#viewmodel-implementation)
5. [User Interface Implementation](#user-interface-implementation)
6. [Putting It All Together](#putting-it-all-together)

## Project Setup

### Create a new Android project:
1. Open Android Studio
2. Create a new project with an Empty Activity
3. Name it "BudgetTrackerApp"
4. Set minimum SDK to API 24 (Android 7.0) or higher

### Add the necessary dependencies in your app-level build.gradle file:

```gradle
plugins {
    id 'com.android.application'
    id 'org.jetbrains.kotlin.android'
    id 'kotlin-kapt'
}

android {
    namespace 'com.example.budgettrackerapp'
    compileSdk 34

    defaultConfig {
        applicationId "com.example.budgettrackerapp"
        minSdk 24
        targetSdk 34
        versionCode 1
        versionName "1.0"

        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
    }

    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_17
        targetCompatibility JavaVersion.VERSION_17
    }
    kotlinOptions {
        jvmTarget = '17'
    }
    buildFeatures {
        viewBinding true
    }
}

dependencies {
    // AndroidX Core Libraries
    implementation 'androidx.core:core-ktx:1.12.0'
    implementation 'androidx.appcompat:appcompat:1.6.1'
    implementation 'com.google.android.material:material:1.11.0'
    implementation 'androidx.constraintlayout:constraintlayout:2.1.4'
    implementation 'androidx.lifecycle:lifecycle-livedata-ktx:2.7.0'
    implementation 'androidx.lifecycle:lifecycle-viewmodel-ktx:2.7.0'
    implementation 'androidx.navigation:navigation-fragment-ktx:2.7.7'
    implementation 'androidx.navigation:navigation-ui-ktx:2.7.7'
    
    // Room components
    implementation "androidx.room:room-runtime:2.6.1"
    kapt "androidx.room:room-compiler:2.6.1"
    implementation "androidx.room:room-ktx:2.6.1"
    
    // Coroutines
    implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-android:1.7.3'
    
    // Image handling
    implementation 'com.github.bumptech.glide:glide:4.16.0'
    
    // Material Components
    implementation 'com.google.android.material:material:1.11.0'
    
    // Testing
    testImplementation 'junit:junit:4.13.2'
    androidTestImplementation 'androidx.test.ext:junit:1.1.5'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.5.1'
}
```

## Database Implementation

### 1. Create the Entity Classes

#### User.kt
```kotlin
package com.example.budgettrackerapp.data.entities

import androidx.room.Entity
import androidx.room.PrimaryKey

@Entity(tableName = "users")
data class User(
    @PrimaryKey(autoGenerate = true)
    val userId: Long = 0,
    val username: String,
    val password: String
)
```

#### Category.kt
```kotlin
package com.example.budgettrackerapp.data.entities

import androidx.room.Entity
import androidx.room.PrimaryKey

@Entity(tableName = "categories")
data class Category(
    @PrimaryKey(autoGenerate = true)
    val categoryId: Long = 0,
    val name: String,
    val userId: Long
)
```

#### Expense.kt
```kotlin
package com.example.budgettrackerapp.data.entities

import androidx.room.Entity
import androidx.room.ForeignKey
import androidx.room.PrimaryKey
import java.util.Date

@Entity(
    tableName = "expenses",
    foreignKeys = [
        ForeignKey(
            entity = Category::class,
            parentColumns = ["categoryId"],
            childColumns = ["categoryId"],
            onDelete = ForeignKey.CASCADE
        ),
        ForeignKey(
            entity = User::class,
            parentColumns = ["userId"],
            childColumns = ["userId"],
            onDelete = ForeignKey.CASCADE
        )
    ]
)
data class Expense(
    @PrimaryKey(autoGenerate = true)
    val expenseId: Long = 0,
    val amount: Double,
    val description: String,
    val date: Long, // Store date as timestamp
    val startTime: Long?, // Optional start time as timestamp
    val endTime: Long?, // Optional end time as timestamp
    val photoPath: String?, // Path to stored photo
    val categoryId: Long,
    val userId: Long
)
```

#### BudgetGoal.kt
```kotlin
package com.example.budgettrackerapp.data.entities

import androidx.room.Entity
import androidx.room.ForeignKey
import androidx.room.PrimaryKey

@Entity(
    tableName = "budget_goals",
    foreignKeys = [
        ForeignKey(
            entity = User::class,
            parentColumns = ["userId"],
            childColumns = ["userId"],
            onDelete = ForeignKey.CASCADE
        )
    ]
)
data class BudgetGoal(
    @PrimaryKey(autoGenerate = true)
    val goalId: Long = 0,
    val minimumGoal: Double,
    val maximumGoal: Double,
    val month: Int, // 1-12
    val year: Int,
    val userId: Long
)
```

### 2. Create Data Access Objects (DAOs)

#### UserDao.kt
```kotlin
package com.example.budgettrackerapp.data.dao

import androidx.room.Dao
import androidx.room.Insert
import androidx.room.Query
import com.example.budgettrackerapp.data.entities.User

@Dao
interface UserDao {
    @Insert
    suspend fun insertUser(user: User): Long
    
    @Query("SELECT * FROM users WHERE username = :username AND password = :password")
    suspend fun getUserByCredentials(username: String, password: String): User?
    
    @Query("SELECT * FROM users WHERE username = :username")
    suspend fun getUserByUsername(username: String): User?
}
```

#### CategoryDao.kt
```kotlin
package com.example.budgettrackerapp.data.dao

import androidx.lifecycle.LiveData
import androidx.room.Dao
import androidx.room.Delete
import androidx.room.Insert
import androidx.room.Query
import androidx.room.Update
import com.example.budgettrackerapp.data.entities.Category

@Dao
interface CategoryDao {
    @Insert
    suspend fun insertCategory(category: Category): Long
    
    @Update
    suspend fun updateCategory(category: Category)
    
    @Delete
    suspend fun deleteCategory(category: Category)
    
    @Query("SELECT * FROM categories WHERE userId = :userId")
    fun getCategoriesByUser(userId: Long): LiveData<List<Category>>
    
    @Query("SELECT * FROM categories WHERE categoryId = :categoryId")
    suspend fun getCategoryById(categoryId: Long): Category?
}
```

#### ExpenseDao.kt
```kotlin
package com.example.budgettrackerapp.data.dao

import androidx.lifecycle.LiveData
import androidx.room.Dao
import androidx.room.Delete
import androidx.room.Insert
import androidx.room.Query
import androidx.room.Update
import com.example.budgettrackerapp.data.entities.Expense

@Dao
interface ExpenseDao {
    @Insert
    suspend fun insertExpense(expense: Expense): Long
    
    @Update
    suspend fun updateExpense(expense: Expense)
    
    @Delete
    suspend fun deleteExpense(expense: Expense)
    
    @Query("SELECT * FROM expenses WHERE userId = :userId AND date BETWEEN :startDate AND :endDate ORDER BY date DESC")
    fun getExpensesByUserAndDateRange(userId: Long, startDate: Long, endDate: Long): LiveData<List<Expense>>
    
    @Query("SELECT SUM(amount) FROM expenses WHERE userId = :userId AND categoryId = :categoryId AND date BETWEEN :startDate AND :endDate")
    suspend fun getTotalAmountByCategory(userId: Long, categoryId: Long, startDate: Long, endDate: Long): Double?
    
    @Query("SELECT * FROM expenses WHERE expenseId = :expenseId")
    suspend fun getExpenseById(expenseId: Long): Expense?
}
```

#### BudgetGoalDao.kt
```kotlin
package com.example.budgettrackerapp.data.dao

import androidx.lifecycle.LiveData
import androidx.room.Dao
import androidx.room.Insert
import androidx.room.OnConflictStrategy
import androidx.room.Query
import androidx.room.Update
import com.example.budgettrackerapp.data.entities.BudgetGoal

@Dao
interface BudgetGoalDao {
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertOrUpdateBudgetGoal(budgetGoal: BudgetGoal): Long
    
    @Update
    suspend fun updateBudgetGoal(budgetGoal: BudgetGoal)
    
    @Query("SELECT * FROM budget_goals WHERE userId = :userId AND month = :month AND year = :year")
    suspend fun getBudgetGoalByMonthYear(userId: Long, month: Int, year: Int): BudgetGoal?
    
    @Query("SELECT * FROM budget_goals WHERE userId = :userId ORDER BY year DESC, month DESC")
    fun getAllBudgetGoalsByUser(userId: Long): LiveData<List<BudgetGoal>>
}
```

### 3. Create the Database class

#### AppDatabase.kt
```kotlin
package com.example.budgettrackerapp.data

import android.content.Context
import androidx.room.Database
import androidx.room.Room
import androidx.room.RoomDatabase
import com.example.budgettrackerapp.data.dao.BudgetGoalDao
import com.example.budgettrackerapp.data.dao.CategoryDao
import com.example.budgettrackerapp.data.dao.ExpenseDao
import com.example.budgettrackerapp.data.dao.UserDao
import com.example.budgettrackerapp.data.entities.BudgetGoal
import com.example.budgettrackerapp.data.entities.Category
import com.example.budgettrackerapp.data.entities.Expense
import com.example.budgettrackerapp.data.entities.User

@Database(
    entities = [User::class, Category::class, Expense::class, BudgetGoal::class],
    version = 1,
    exportSchema = false
)
abstract class AppDatabase : RoomDatabase() {
    abstract fun userDao(): UserDao
    abstract fun categoryDao(): CategoryDao
    abstract fun expenseDao(): ExpenseDao
    abstract fun budgetGoalDao(): BudgetGoalDao

    companion object {
        @Volatile
        private var INSTANCE: AppDatabase? = null

        fun getDatabase(context: Context): AppDatabase {
            return INSTANCE ?: synchronized(this) {
                val instance = Room.databaseBuilder(
                    context.applicationContext,
                    AppDatabase::class.java,
                    "budget_tracker_database"
                ).build()
                INSTANCE = instance
                instance
            }
        }
    }
}
```

## Repository Implementation

### 1. Create Repository Classes

#### UserRepository.kt
```kotlin
package com.example.budgettrackerapp.repository

import com.example.budgettrackerapp.data.dao.UserDao
import com.example.budgettrackerapp.data.entities.User

class UserRepository(private val userDao: UserDao) {
    
    suspend fun registerUser(username: String, password: String): Result<Long> {
        return try {
            val existingUser = userDao.getUserByUsername(username)
            if (existingUser != null) {
                Result.failure(Exception("Username already exists"))
            } else {
                val userId = userDao.insertUser(User(username = username, password = password))
                Result.success(userId)
            }
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
    
    suspend fun loginUser(username: String, password: String): Result<User> {
        return try {
            val user = userDao.getUserByCredentials(username, password)
            if (user != null) {
                Result.success(user)
            } else {
                Result.failure(Exception("Invalid username or password"))
            }
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}
```

#### CategoryRepository.kt
```kotlin
package com.example.budgettrackerapp.repository

import androidx.lifecycle.LiveData
import com.example.budgettrackerapp.data.dao.CategoryDao
import com.example.budgettrackerapp.data.entities.Category

class CategoryRepository(private val categoryDao: CategoryDao) {
    
    fun getUserCategories(userId: Long): LiveData<List<Category>> {
        return categoryDao.getCategoriesByUser(userId)
    }
    
    suspend fun addCategory(name: String, userId: Long): Result<Long> {
        return try {
            val categoryId = categoryDao.insertCategory(Category(name = name, userId = userId))
            Result.success(categoryId)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
    
    suspend fun updateCategory(category: Category): Result<Unit> {
        return try {
            categoryDao.updateCategory(category)
            Result.success(Unit)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
    
    suspend fun deleteCategory(category: Category): Result<Unit> {
        return try {
            categoryDao.deleteCategory(category)
            Result.success(Unit)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
    
    suspend fun getCategoryById(categoryId: Long): Result<Category> {
        return try {
            val category = categoryDao.getCategoryById(categoryId)
            if (category != null) {
                Result.success(category)
            } else {
                Result.failure(Exception("Category not found"))
            }
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}
```

#### ExpenseRepository.kt
```kotlin
package com.example.budgettrackerapp.repository

import androidx.lifecycle.LiveData
import com.example.budgettrackerapp.data.dao.ExpenseDao
import com.example.budgettrackerapp.data.entities.Expense
import java.util.Calendar
import java.util.Date

class ExpenseRepository(private val expenseDao: ExpenseDao) {
    
    fun getExpensesForPeriod(userId: Long, startDate: Long, endDate: Long): LiveData<List<Expense>> {
        return expenseDao.getExpensesByUserAndDateRange(userId, startDate, endDate)
    }
    
    suspend fun addExpense(
        amount: Double,
        description: String,
        date: Long,
        startTime: Long?,
        endTime: Long?,
        photoPath: String?,
        categoryId: Long,
        userId: Long
    ): Result<Long> {
        return try {
            val expense = Expense(
                amount = amount,
                description = description,
                date = date,
                startTime = startTime,
                endTime = endTime,
                photoPath = photoPath,
                categoryId = categoryId,
                userId = userId
            )
            val expenseId = expenseDao.insertExpense(expense)
            Result.success(expenseId)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
    
    suspend fun updateExpense(expense: Expense): Result<Unit> {
        return try {
            expenseDao.updateExpense(expense)
            Result.success(Unit)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
    
    suspend fun deleteExpense(expense: Expense): Result<Unit> {
        return try {
            expenseDao.deleteExpense(expense)
            Result.success(Unit)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
    
    suspend fun getTotalAmountByCategory(
        userId: Long,
        categoryId: Long,
        startDate: Long,
        endDate: Long
    ): Result<Double> {
        return try {
            val total = expenseDao.getTotalAmountByCategory(userId, categoryId, startDate, endDate) ?: 0.0
            Result.success(total)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
    
    suspend fun getExpenseById(expenseId: Long): Result<Expense> {
        return try {
            val expense = expenseDao.getExpenseById(expenseId)
            if (expense != null) {
                Result.success(expense)
            } else {
                Result.failure(Exception("Expense not found"))
            }
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}
```

#### BudgetGoalRepository.kt
```kotlin
package com.example.budgettrackerapp.repository

import androidx.lifecycle.LiveData
import com.example.budgettrackerapp.data.dao.BudgetGoalDao
import com.example.budgettrackerapp.data.entities.BudgetGoal

class BudgetGoalRepository(private val budgetGoalDao: BudgetGoalDao) {
    
    suspend fun setBudgetGoal(
        minimumGoal: Double,
        maximumGoal: Double,
        month: Int,
        year: Int,
        userId: Long
    ): Result<Long> {
        return try {
            val budgetGoal = BudgetGoal(
                minimumGoal = minimumGoal,
                maximumGoal = maximumGoal,
                month = month,
                year = year,
                userId = userId
            )
            val goalId = budgetGoalDao.insertOrUpdateBudgetGoal(budgetGoal)
            Result.success(goalId)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
    
    suspend fun updateBudgetGoal(budgetGoal: BudgetGoal): Result<Unit> {
        return try {
            budgetGoalDao.updateBudgetGoal(budgetGoal)
            Result.success(Unit)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
    
    suspend fun getBudgetGoalByMonthYear(userId: Long, month: Int, year: Int): Result<BudgetGoal?> {
        return try {
            val budgetGoal = budgetGoalDao.getBudgetGoalByMonthYear(userId, month, year)
            Result.success(budgetGoal)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
    
    fun getAllBudgetGoalsByUser(userId: Long): LiveData<List<BudgetGoal>> {
        return budgetGoalDao.getAllBudgetGoalsByUser(userId)
    }
}
```

## ViewModel Implementation

### 1. Create Session Manager for user state

#### SessionManager.kt
```kotlin
package com.example.budgettrackerapp.utils

import android.content.Context
import android.content.SharedPreferences
import androidx.security.crypto.EncryptedSharedPreferences
import androidx.security.crypto.MasterKey

class SessionManager(context: Context) {
    
    private val masterKey = MasterKey.Builder(context)
        .setKeyScheme(MasterKey.KeyScheme.AES256_GCM)
        .build()
    
    private val sharedPreferences: SharedPreferences = EncryptedSharedPreferences.create(
        context,
        "user_session",
        masterKey,
        EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
        EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
    )
    
    companion object {
        private const val KEY_USER_ID = "user_id"
        private const val KEY_USERNAME = "username"
        private const val KEY_IS_LOGGED_IN = "is_logged_in"
    }
    
    fun saveUserSession(userId: Long, username: String) {
        val editor = sharedPreferences.edit()
        editor.putLong(KEY_USER_ID, userId)
        editor.putString(KEY_USERNAME, username)
        editor.putBoolean(KEY_IS_LOGGED_IN, true)
        editor.apply()
    }
    
    fun getUserId(): Long {
        return sharedPreferences.getLong(KEY_USER_ID, -1)
    }
    
    fun getUsername(): String? {
        return sharedPreferences.getString(KEY_USERNAME, null)
    }
    
    fun isLoggedIn(): Boolean {
        return sharedPreferences.getBoolean(KEY_IS_LOGGED_IN, false)
    }
    
    fun clearSession() {
        val editor = sharedPreferences.edit()
        editor.clear()
        editor.apply()
    }
}
```

### 2. Create ViewModels

#### AuthViewModel.kt
```kotlin
package com.example.budgettrackerapp.viewmodels

import androidx.lifecycle.LiveData
import androidx.lifecycle.MutableLiveData
import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.example.budgettrackerapp.data.entities.User
import com.example.budgettrackerapp.repository.UserRepository
import kotlinx.coroutines.launch

class AuthViewModel(private val userRepository: UserRepository) : ViewModel() {
    
    private val _loginResult = MutableLiveData<Result<User>>()
    val loginResult: LiveData<Result<User>> = _loginResult
    
    private val _registerResult = MutableLiveData<Result<Long>>()
    val registerResult: LiveData<Result<Long>> = _registerResult
    
    fun login(username: String, password: String) {
        viewModelScope.launch {
            val result = userRepository.loginUser(username, password)
            _loginResult.postValue(result)
        }
    }
    
    fun register(username: String, password: String) {
        viewModelScope.launch {
            val result = userRepository.registerUser(username, password)
            _registerResult.postValue(result)
        }
    }
}
```

#### CategoryViewModel.kt
```kotlin
package com.example.budgettrackerapp.viewmodels

import androidx.lifecycle.LiveData
import androidx.lifecycle.MutableLiveData
import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.example.budgettrackerapp.data.entities.Category
import com.example.budgettrackerapp.repository.CategoryRepository
import kotlinx.coroutines.launch

class CategoryViewModel(private val categoryRepository: CategoryRepository) : ViewModel() {
    
    private val _categoryOperationResult = MutableLiveData<Result<Any>>()
    val categoryOperationResult: LiveData<Result<Any>> = _categoryOperationResult
    
    fun getUserCategories(userId: Long): LiveData<List<Category>> {
        return categoryRepository.getUserCategories(userId)
    }
    
    fun addCategory(name: String, userId: Long) {
        viewModelScope.launch {
            val result = categoryRepository.addCategory(name, userId)
            _categoryOperationResult.postValue(result)
        }
    }
    
    fun updateCategory(category: Category) {
        viewModelScope.launch {
            val result = categoryRepository.updateCategory(category)
            _categoryOperationResult.postValue(result)
        }
    }
    
    fun deleteCategory(category: Category) {
        viewModelScope.launch {
            val result = categoryRepository.deleteCategory(category)
            _categoryOperationResult.postValue(result)
        }
    }
}
```

#### ExpenseViewModel.kt
```kotlin
package com.example.budgettrackerapp.viewmodels

import androidx.lifecycle.LiveData
import androidx.lifecycle.MutableLiveData
import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.example.budgettrackerapp.data.entities.Expense
import com.example.budgettrackerapp.repository.ExpenseRepository
import kotlinx.coroutines.launch
import java.util.Calendar
import java.util.Date

class ExpenseViewModel(private val expenseRepository: ExpenseRepository) : ViewModel() {
    
    private val _expenseOperationResult = MutableLiveData<Result<Any>>()
    val expenseOperationResult: LiveData<Result<Any>> = _expenseOperationResult
    
    private val _categoryTotalAmount = MutableLiveData<Map<Long, Double>>()
    val categoryTotalAmount: LiveData<Map<Long, Double>> = _categoryTotalAmount
    
    fun getExpensesForPeriod(userId: Long, startDate: Long, endDate: Long): LiveData<List<Expense>> {
        return expenseRepository.getExpensesForPeriod(userId, startDate, endDate)
    }
    
    fun addExpense(
        amount: Double,
        description: String,
        date: Long,
        startTime: Long?,
        endTime: Long?,
        photoPath: String?,
        categoryId: Long,
        userId: Long
    ) {
        viewModelScope.launch {
            val result = expenseRepository.addExpense(
                amount, description, date, startTime, endTime, photoPath, categoryId, userId
            )
            _expenseOperationResult.postValue(result)
        }
    }
    
    fun updateExpense(expense: Expense) {
        viewModelScope.launch {
            val result = expenseRepository.updateExpense(expense)
            _expenseOperationResult.postValue(result)
        }
    }
    
    fun deleteExpense(expense: Expense) {
        viewModelScope.launch {
            val result = expenseRepository.deleteExpense(expense)
            _expenseOperationResult.postValue(result)
        }
    }
    
    fun calculateTotalAmountByCategories(
        userId: Long,
        categoryIds: List<Long>,
        startDate: Long,
        endDate: Long
    ) {
        viewModelScope.launch {
            val totalsMap = mutableMapOf<Long, Double>()
            
            for (categoryId in categoryIds) {
                val result = expenseRepository.getTotalAmountByCategory(
                    userId, categoryId, startDate, endDate
                )
                if (result.isSuccess) {
                    totalsMap[categoryId] = result.getOrNull() ?: 0.0
                }
            }
            
            _categoryTotalAmount.postValue(totalsMap)
        }
    }
}
```

#### BudgetGoalViewModel.kt
```kotlin
package com.example.budgettrackerapp.viewmodels

import androidx.lifecycle.LiveData
import androidx.lifecycle.MutableLiveData
import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.example.budgettrackerapp.data.entities.BudgetGoal
import com.example.budgettrackerapp.repository.BudgetGoalRepository
import kotlinx.coroutines.launch

class BudgetGoalViewModel(private val budgetGoalRepository: BudgetGoalRepository) : ViewModel() {
    
    private val _budgetGoalOperationResult = MutableLiveData<Result<Any>>()
    val budgetGoalOperationResult: LiveData<Result<Any>> = _budgetGoalOperationResult
    
    private val _currentBudgetGoal = MutableLiveData<BudgetGoal?>()
    val currentBudgetGoal: LiveData<BudgetGoal?> = _currentBudgetGoal
    
    fun setBudgetGoal(minimumGoal: Double, maximumGoal: Double, month: Int, year: Int, userId: Long) {
        viewModelScope.launch {
            val result = budgetGoalRepository.setBudgetGoal(minimumGoal, maximumGoal, month, year, userId)
            _budgetGoalOperationResult.postValue(result)
        }
    }
    
    fun getAllBudgetGoalsByUser(userId: Long): LiveData<List<BudgetGoal>> {
        return budgetGoalRepository.getAllBudgetGoalsByUser(userId)
    }
    
    fun getBudgetGoalForMonth(userId: Long, month: Int, year: Int) {
        viewModelScope.launch {
            val result = budgetGoalRepository.getBudgetGoalByMonthYear(userId, month, year)
            if (result.isSuccess) {
                _currentBudgetGoal.postValue(result.getOrNull())
            } else {
                _currentBudgetGoal.postValue(null)
            }
        }
    }
}
```

### 3. Create ViewModelFactory

#### ViewModelFactory.kt
```kotlin
package com.example.budgettrackerapp.viewmodels

import androidx.lifecycle.ViewModel
import androidx.lifecycle.ViewModelProvider
import com.example.budgettrackerapp.repository.BudgetGoalRepository
import com.example.budgettrackerapp.repository.CategoryRepository
import com.example.budgettrackerapp.repository.ExpenseRepository
import com.example.budgettrackerapp.repository.UserRepository

class ViewModelFactory(
    private val userRepository: UserRepository? = null,
    private val categoryRepository: CategoryRepository? = null,
    private val expenseRepository: ExpenseRepository? = null,
    private val budgetGoalRepository: BudgetGoalRepository? = null
) : ViewModelProvider.Factory {
    
    @Suppress("UNCHECKED_CAST")
    override fun <T : ViewModel> create(modelClass: Class<T>): T {
        return when {
            modelClass.isAssignableFrom(AuthViewModel::class.java) -> {
                userRepository?.let { AuthViewModel(it) as T }
                    ?: throw IllegalArgumentException("AuthViewModel requires UserRepository")
            }
            modelClass.isAssignableFrom(CategoryViewModel::class.java) -> {
                categoryRepository?.let { CategoryViewModel(it) as T }
                    ?: throw IllegalArgumentException("CategoryViewModel requires CategoryRepository")
            }
            modelClass.isAssignableFrom(ExpenseViewModel::class.java) -> {
                expenseRepository?.let { ExpenseViewModel(it) as T }
                    ?: throw IllegalArgumentException("ExpenseViewModel requires ExpenseRepository")
            }
            modelClass.isAssignableFrom(BudgetGoalViewModel::class.java) -> {
                budgetGoalRepository?.let { BudgetGoalViewModel(it) as T }
                    ?: throw IllegalArgumentException("BudgetGoalViewModel requires BudgetGoalRepository")
            }
            else -> throw IllegalArgumentException("Unknown ViewModel class: ${modelClass.name}")
            }
        }
    }
}
```

## User Interface Implementation

### 1. Create navigation graph

Create a new resource file: `res/navigation/nav_graph.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<navigation xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/nav_graph"
    app:startDestination="@id/loginFragment">

    <fragment
        android:id="@+id/loginFragment"
        android:name="com.example.budgettrackerapp.ui.auth.LoginFragment"
        android:label="Login"
        tools:layout="@layout/fragment_login">
        <action
            android:id="@+id/action_loginFragment_to_registerFragment"
            app:destination="@id/registerFragment" />
        <action
            android:id="@+id/action_loginFragment_to_homeFragment"
            app:destination="@id/homeFragment"
            app:popUpTo="@id/loginFragment"
            app:popUpToInclusive="true" />
    </fragment>

    <fragment
        android:id="@+id/registerFragment"
        android:name="com.example.budgettrackerapp.ui.auth.RegisterFragment"
        android:label="Register"
        tools:layout="@layout/fragment_register">
        <action
            android:id="@+id/action_registerFragment_to_loginFragment"
            app:destination="@id/loginFragment"
            app:popUpTo="@id/loginFragment"
            app:popUpToInclusive="true" />
    </fragment>

    <fragment
        android:id="@+id/homeFragment"
        android:name="com.example.budgettrackerapp.ui.home.HomeFragment"
        android:label="Home"
        tools:layout="@layout/fragment_home">
        <action
            android:id="@+id/action_homeFragment_to_addExpenseFragment"
            app:destination="@id/addExpenseFragment" />
        <action
            android:id="@+id/action_homeFragment_to_categoriesFragment"
            app:destination="@id/categoriesFragment" />
        <action
            android:id="@+id/action_homeFragment_to_budgetGoalFragment"
            app:destination="@id/budgetGoalFragment" />
        <action
            android:id="@+id/action_homeFragment_to_transactionsFragment"
            app:destination="@id/transactionsFragment" />
        <action
            android:id="@+id/action_homeFragment_to_dashboardFragment"
            app:destination="@id/dashboardFragment" />
    </fragment>

    <fragment
        android:id="@+id/addExpenseFragment"
        android:name="com.example.budgettrackerapp.ui.expenses.AddExpenseFragment"
        android:label="Add Expense"
        tools:layout="@layout/fragment_add_expense">
        <action
            android:id="@+id/action_addExpenseFragment_to_homeFragment"
            app:destination="@id/homeFragment"
            app:popUpTo="@id/homeFragment"
            app:popUpToInclusive="true" />
    </fragment>

    <fragment
        android:id="@+id/categoriesFragment"
        android:name="com.example.budgettrackerapp.ui.categories.CategoriesFragment"
        android:label="Categories"
        tools:layout="@layout/fragment_categories">
        <action
            android:id="@+id/action_categoriesFragment_to_homeFragment"
            app:destination="@id/homeFragment"
            app:popUpTo="@id/homeFragment"
            app:popUpToInclusive="true" />
    </fragment>

    <fragment
        android:id="@+id/budgetGoalFragment"
        android:name="com.example.budgettrackerapp.ui.budget.BudgetGoalFragment"
        android:label="Budget Goals"
        tools:layout="@layout/fragment_budget_goal">
        <action
            android:id="@+id/action_budgetGoalFragment_to_homeFragment"
            app:destination="@id/homeFragment"
            app:popUpTo="@id/homeFragment"
            app:popUpToInclusive="true" />
    </fragment>

    <fragment
        android:id="@+id/transactionsFragment"
        android:name="com.example.budgettrackerapp.ui.transactions.TransactionsFragment"
        android:label="Transactions"
        tools:layout="@layout/fragment_transactions">
        <action
            android:id="@+id/action_transactionsFragment_to_homeFragment"
            app:destination="@id/homeFragment"
            app:popUpTo="@id/homeFragment"
            app:popUpToInclusive="true" />
        <action
            android:id="@+id/action_transactionsFragment_to_expenseDetailFragment"
            app:destination="@id/expenseDetailFragment" />
    </fragment>

    <fragment
        android:id="@+id/expenseDetailFragment"
        android:name="com.example.budgettrackerapp.ui.transactions.ExpenseDetailFragment"
        android:label="Expense Detail"
        tools:layout="@layout/fragment_expense_detail">
        <argument
            android:name="expenseId"
            app:argType="long" />
        <action
            android:id="@+id/action_expenseDetailFragment_to_transactionsFragment"
            app:destination="@id/transactionsFragment"
            app:popUpTo="@id/transactionsFragment"
            app:popUpToInclusive="true" />
    </fragment>

    <fragment
        android:id="@+id/dashboardFragment"
        android:name="com.example.budgettrackerapp.ui.dashboard.DashboardFragment"
        android:label="Dashboard"
        tools:layout="@layout/fragment_dashboard">
        <action
            android:id="@+id/action_dashboardFragment_to_homeFragment"
            app:destination="@id/homeFragment"
            app:popUpTo="@id/homeFragment"
            app:popUpToInclusive="true" />
        <action
            android:id="@+id/action_dashboardFragment_to_categoryDetailFragment"
            app:destination="@id/categoryDetailFragment" />
    </fragment>

    <fragment
        android:id="@+id/categoryDetailFragment"
        android:name="com.example.budgettrackerapp.ui.dashboard.CategoryDetailFragment"
        android:label="Category Detail"
        tools:layout="@layout/fragment_category_detail">
        <argument
            android:name="categoryId"
            app:argType="long" />
        <action
            android:id="@+id/action_categoryDetailFragment_to_dashboardFragment"
            app:destination="@id/dashboardFragment"
            app:popUpTo="@id/dashboardFragment"
            app:popUpToInclusive="true" />
    </fragment>
</navigation>
```

### 2. Update MainActivity

#### MainActivity.kt
```kotlin
package com.example.budgettrackerapp

import android.os.Bundle
import androidx.appcompat.app.AppCompatActivity
import androidx.navigation.fragment.NavHostFragment
import androidx.navigation.ui.setupWithNavController
import com.example.budgettrackerapp.databinding.ActivityMainBinding
import com.example.budgettrackerapp.utils.SessionManager

class MainActivity : AppCompatActivity() {

    private lateinit var binding: ActivityMainBinding
    private lateinit var sessionManager: SessionManager

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        sessionManager = SessionManager(this)

        val navHostFragment = supportFragmentManager
            .findFragmentById(R.id.nav_host_fragment) as NavHostFragment
        val navController = navHostFragment.navController

        // Setup bottom navigation
        binding.bottomNavigation.setupWithNavController(navController)

        // Hide bottom navigation in auth screens
        navController.addOnDestinationChangedListener { _, destination, _ ->
            when (destination.id) {
                R.id.loginFragment, R.id.registerFragment -> binding.bottomNavigation.hide()
                else -> binding.bottomNavigation.show()
            }
        }
    }
}
```

### 3. Create Layout Files

#### activity_main.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <androidx.fragment.app.FragmentContainerView
        android:id="@+id/nav_host_fragment"
        android:name="androidx.navigation.fragment.NavHostFragment"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:defaultNavHost="true"
        app:layout_constraintBottom_toTopOf="@+id/bottom_navigation"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:navGraph="@navigation/nav_graph" />

    <com.google.android.material.bottomnavigation.BottomNavigationView
        android:id="@+id/bottom_navigation"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:menu="@menu/bottom_nav_menu" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

#### fragment_login.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="16dp"
    tools:context=".ui.auth.LoginFragment">

    <TextView
        android:id="@+id/tv_login_title"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="64dp"
        android:text="Login"
        android:textSize="24sp"
        android:textStyle="bold"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <com.google.android.material.textfield.TextInputLayout
        android:id="@+id/til_username"
        style="@style/Widget.MaterialComponents.TextInputLayout.OutlinedBox"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginTop="32dp"
        android:hint="Username"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/tv_login_title">

        <com.google.android.material.textfield.TextInputEditText
            android:id="@+id/et_username"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:inputType="text" />
    </com.google.android.material.textfield.TextInputLayout>

    <com.google.android.material.textfield.TextInputLayout
        android:id="@+id/til_password"
        style="@style/Widget.MaterialComponents.TextInputLayout.OutlinedBox"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        android:hint="Password"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/til_username"
        app:passwordToggleEnabled="true">

        <com.google.android.material.textfield.TextInputEditText
            android:id="@+id/et_password"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:inputType="textPassword" />
    </com.google.android.material.textfield.TextInputLayout>

    <Button
        android:id="@+id/btn_login"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginTop="24dp"
        android:text="Login"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/til_password" />

    <TextView
        android:id="@+id/tv_register_link"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        android:text="Don't have an account? Register here"
        android:textColor="@color/design_default_color_primary"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/btn_login" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

#### fragment_register.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="16dp"
    tools:context=".ui.auth.RegisterFragment">

    <TextView
        android:id="@+id/tv_register_title"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="64dp"
        android:text="Create New Account"
        android:textSize="24sp"
        android:textStyle="bold"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <com.google.android.material.textfield.TextInputLayout
        android:id="@+id/til_username"
        style="@style/Widget.MaterialComponents.TextInputLayout.OutlinedBox"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginTop="32dp"
        android:hint="Username"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/tv_register_title">

        <com.google.android.material.textfield.TextInputEditText
            android:id="@+id/et_username"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:inputType="text" />
    </com.google.android.material.textfield.TextInputLayout>

    <com.google.android.material.textfield.TextInputLayout
        android:id="@+id/til_password"
        style="@style/Widget.MaterialComponents.TextInputLayout.OutlinedBox"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        android:hint="Password"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/til_username"
        app:passwordToggleEnabled="true">

        <com.google.android.material.textfield.TextInputEditText
            android:id="@+id/et_password"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:inputType="textPassword" />
    </com.google.android.material.textfield.TextInputLayout>

    <com.google.android.material.textfield.TextInputLayout
        android:id="@+id/til_confirm_password"
        style="@style/Widget.MaterialComponents.TextInputLayout.OutlinedBox"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        android:hint="Confirm Password"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/til_password"
        app:passwordToggleEnabled="true">

        <com.google.android.material.textfield.TextInputEditText
            android:id="@+id/et_confirm_password"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:inputType="textPassword" />
    </com.google.android.material.textfield.TextInputLayout>

    <Button
        android:id="@+id/btn_register"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginTop="24dp"
        android:text="Register"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/til_confirm_password" />

    <TextView
        android:id="@+id/tv_login_link"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        android:text="Already have an account? Login here"
        android:textColor="@color/design_default_color_primary"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/btn_register" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

#### fragment_home.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="16dp"
    tools:context=".ui.home.HomeFragment">

    <TextView
        android:id="@+id/tv_welcome"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        android:text="Welcome Back!"
        android:textSize="24sp"
        android:textStyle="bold"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <com.google.android.material.card.MaterialCardView
        android:id="@+id/card_stats"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginTop="24dp"
        app:cardCornerRadius="8dp"
        app:cardElevation="4dp"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/tv_welcome">

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:padding="16dp">

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="Current Month Overview"
                android:textSize="18sp"
                android:textStyle="bold" />

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="16dp"
                android:orientation="horizontal">

                <LinearLayout
                    android:layout_width="0dp"
                    android:layout_height="wrap_content"
                    android:layout_weight="1"
                    android:orientation="vertical">

                    <TextView
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:text="Total Spent"
                        android:textSize="14sp" />

                    <TextView
                        android:id="@+id/tv_total_spent"
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="4dp"
                        android:text="$0.00"
                        android:textSize="18sp"
                        android:textStyle="bold" />
                </LinearLayout>

                <LinearLayout
                    android:layout_width="0dp"
                    android:layout_height="wrap_content"
                    android:layout_weight="1"
                    android:orientation="vertical">

                    <TextView
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:text="Budget Goal"
                        android:textSize="14sp" />

                    <TextView
                        android:id="@+id/tv_budget_goal"
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:layout_marginTop="4dp"
                        android:text="$0.00 - $0.00"
                        android:textSize="18sp"
                        android:textStyle="bold" />
                </LinearLayout>
            </LinearLayout>

            <ProgressBar
                android:id="@+id/progress_budget"
                style="?android:attr/progressBarStyleHorizontal"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="16dp"
                android:max="100"
                android:progress="0" />
        </LinearLayout>
    </com.google.android.material.card.MaterialCardView>

    <GridLayout
        android:layout_width="0dp"
        android:layout_height="0dp"
        android:layout_marginTop="24dp"
        android:columnCount="2"
        android:rowCount="3"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/card_stats">

        <com.google.android.material.card.MaterialCardView
            android:id="@+id/card_add_expense"
            android:layout_width="0dp"
            android:layout_height="0dp"
            android:layout_rowWeight="1"
            android:layout_columnWeight="1"
            android:layout_margin="8dp"
            app:cardCornerRadius="8dp"
            app:cardElevation="4dp">

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:gravity="center"
                android:orientation="vertical"
                android:padding="16dp">

                <ImageView
                    android:layout_width="48dp"
                    android:layout_height="48dp"
                    android:src="@android:drawable/ic_input_add"
                    app:tint="@color/design_default_color_primary" />

                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="8dp"
                    android:text="Add Expense"
                    android:textSize="16sp"
                    android:textStyle="bold" />
            </LinearLayout>
        </com.google.android.material.card.MaterialCardView>

        <com.google.android.material.card.MaterialCardView
            android:id="@+id/card_categories"
            android:layout_width="0dp"
            android:layout_height="0dp"
            android:layout_rowWeight="1"
            android:layout_columnWeight="1"
            android:layout_margin="8dp"
            app:cardCornerRadius="8dp"
            app:cardElevation="4dp">

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:gravity="center"
                android:orientation="vertical"
                android:padding="16dp">

                <ImageView
                    android:layout_width="48dp"
                    android:layout_height="48dp"
                    android:src="@android:drawable/ic_menu_sort_by_size"
                    app:tint="@color/design_default_color_primary" />

                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="8dp"
                    android:text="Categories"
                    android:textSize="16sp"
                    android:textStyle="bold" />
            </LinearLayout>
        </com.google.android.material.card.MaterialCardView>

        <com.google.android.material.card.MaterialCardView
            android:id="@+id/card_transactions"
            android:layout_width="0dp"
            android:layout_height="0dp"
            android:layout_rowWeight="1"
            android:layout_columnWeight="1"
            android:layout_margin="8dp"
            app:cardCornerRadius="8dp"
            app:cardElevation="4dp">

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:gravity="center"
                android:orientation="vertical"
                android:padding="16dp">

                <ImageView
                    android:layout_width="48dp"
                    android:layout_height="48dp"
                    android:src="@android:drawable/ic_menu_recent_history"
                    app:tint="@color/design_default_color_primary" />

                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="8dp"
                    android:text="Transactions"
                    android:textSize="16sp"
                    android:textStyle="bold" />
            </LinearLayout>
        </com.google.android.material.card.MaterialCardView>

        <com.google.android.material.card.MaterialCardView
            android:id="@+id/card_budget_goal"
            android:layout_width="0dp"
            android:layout_height="0dp"
            android:layout_rowWeight="1"
            android:layout_columnWeight="1"
            android:layout_margin="8dp"
            app:cardCornerRadius="8dp"
            app:cardElevation="4dp">

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:gravity="center"
                android:orientation="vertical"
                android:padding="16dp">

                <ImageView
                    android:layout_width="48dp"
                    android:layout_height="48dp"
                    android:src="@android:drawable/ic_menu_edit"
                    app:tint="@color/design_default_color_primary" />

                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="8dp"
                    android:text="Set Budget Goal"
                    android:textSize="16sp"
                    android:textStyle="bold" />
            </LinearLayout>
        </com.google.android.material.card.MaterialCardView>

        <com.google.android.material.card.MaterialCardView
            android:id="@+id/card_dashboard"
            android:layout_width="0dp"
            android:layout_height="0dp"
            android:layout_rowWeight="1"
            android:layout_columnWeight="1"
            android:layout_margin="8dp"
            app:cardCornerRadius="8dp"
            app:cardElevation="4dp">

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:gravity="center"
                android:orientation="vertical"
                android:padding="16dp">

                <ImageView
                    android:layout_width="48dp"
                    android:layout_height="48dp"
                    android:src="@android:drawable/ic_menu_gallery"
                    app:tint="@color/design_default_color_primary" />

                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="8dp"
                    android:text="Dashboard"
                    android:textSize="16sp"
                    android:textStyle="bold" />
            </LinearLayout>
        </com.google.android.material.card.MaterialCardView>

        <com.google.android.material.card.MaterialCardView
            android:id="@+id/card_logout"
            android:layout_width="0dp"
            android:layout_height="0dp"
            android:layout_rowWeight="1"
            android:layout_columnWeight="1"
            android:layout_margin="8dp"
            app:cardCornerRadius="8dp"
            app:cardElevation="4dp">

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:gravity="center"
                android:orientation="vertical"
                android:padding="16dp">

                <ImageView
                    android:layout_width="48dp"
                    android:layout_height="48dp"
                    android:src="@android:drawable/ic_lock_power_off"
                    app:tint="@android:color/holo_red_dark" />

                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_marginTop="8dp"
                    android:text="Logout"
                    android:textSize="16sp"
                    android:textStyle="bold" />
            </LinearLayout>
        </com.google.android.material.card.MaterialCardView>
    </GridLayout>
</androidx.constraintlayout.widget.ConstraintLayout>
```

#### fragment_add_expense.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.core.widget.NestedScrollView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fillViewport="true"
    tools:context=".ui.expenses.AddExpenseFragment">

    <androidx.constraintlayout.widget.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:padding="16dp">

        <TextView
            android:id="@+id/tv_add_expense_title"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Add New Expense"
            android:textSize="24sp"
            android:textStyle="bold"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="parent" />

        <com.google.android.material.textfield.TextInputLayout
            android:id="@+id/til_amount"
            style="@style/Widget.MaterialComponents.TextInputLayout.OutlinedBox"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_marginTop="24dp"
            android:hint="Amount"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toBottomOf="@+id/tv_add_expense_title">

            <com.google.android.material.textfield.TextInputEditText
                android:id="@+id/et_amount"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:inputType="numberDecimal" />
        </com.google.android.material.textfield.TextInputLayout>

        <com.google.android.material.textfield.TextInputLayout
            android:id="@+id/til_description"
            style="@style/Widget.MaterialComponents.TextInputLayout.OutlinedBox"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_marginTop="16dp"
            android:hint="Description"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toBottomOf="@+id/til_amount">

            <com.google.android.material.textfield.TextInputEditText
                android:id="@+id/et_description"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:inputType="text" />
        </com.google.android.material.textfield.TextInputLayout>

        <com.google.android.material.textfield.TextInputLayout
            android:id="@+id/til_category"
            style="@style/Widget.MaterialComponents.TextInputLayout.OutlinedBox.ExposedDropdownMenu"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_marginTop="16dp"
            android:hint="Category"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toBottomOf="@+id/til_description">

            <AutoCompleteTextView
                android:id="@+id/spinner_category"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:inputType="none" />
        </com.google.android.material.textfield.TextInputLayout>

        <TextView
            android:id="@+id/tv_date_label"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginTop="16dp"
            android:text="Date:"
            android:textSize="16sp"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toBottomOf="@+id/til_category" />

        <Button
            android:id="@+id/btn_date_picker"
            style="@style/Widget.MaterialComponents.Button.OutlinedButton"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_marginStart="8dp"
            android:text="Select Date"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toEndOf="@+id/tv_date_label"
            app:layout_constraintTop_toTopOf="@+id/tv_date_label" />

        <TextView
            android:id="@+id/tv_time_label"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginTop="16dp"
            android:text="Time (Optional):"
            android:textSize="16sp"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toBottomOf="@+id/btn_date_picker" />

        <LinearLayout
            android:id="@+id/layout_time_buttons"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_marginStart="8dp"
            android:orientation="horizontal"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toEndOf="@+id/tv_time_label"
            app:layout_constraintTop_toTopOf="@+id/tv_time_label">

            <Button
                android:id="@+id/btn_start_time_picker"
                style="@style/Widget.MaterialComponents.Button.OutlinedButton"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_marginEnd="4dp"
                android:layout_weight="1"
                android:text="Start Time" />

            <Button
                android:id="@+id/btn_end_time_picker"
                style="@style/Widget.MaterialComponents.Button.OutlinedButton"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_marginStart="4dp"
                android:layout_weight="1"
                android:text="End Time" />
        </LinearLayout>

        <LinearLayout
            android:id="@+id/layout_photo"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_marginTop="16dp"
            android:orientation="vertical"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toBottomOf="@+id/layout_time_buttons">

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="Add Photo (Optional):"
                android:textSize="16sp" />

            <Button
                android:id="@+id/btn_add_photo"
                style="@style/Widget.MaterialComponents.Button.OutlinedButton"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="8dp"
                android:text="Take Photo" />

            <ImageView
                android:id="@+id/iv_expense_photo"
                android:layout_width="match_parent"
                android:layout_height="200dp"
                android:layout_marginTop="8dp"
                android:background="@android:color/darker_gray"
                android:scaleType="centerCrop"
                android:visibility="gone" />
        </LinearLayout>

        <Button
            android:id="@+id/btn_save_expense"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_marginTop="24dp"
            android:text="Save Expense"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toBottomOf="@+id/layout_photo" />

    </androidx.constraintlayout.widget.ConstraintLayout>
</androidx.core.widget.NestedScrollView>
```

### 4. Implement Fragment Classes

#### LoginFragment.kt
```kotlin
package com.example.budgettrackerapp.ui.auth

import android.os.Bundle
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.Toast
import androidx.fragment.app.Fragment
import androidx.lifecycle.ViewModelProvider
import androidx.navigation.fragment.findNavController
import com.example.budgettrackerapp.R
import com.example.budgettrackerapp.data.AppDatabase
import com.example.budgettrackerapp.databinding.FragmentLoginBinding
import com.example.budgettrackerapp.repository.UserRepository
import com.example.budgettrackerapp.utils.SessionManager
import com.example.budgettrackerapp.viewmodels.AuthViewModel
import com.example.budgettrackerapp.viewmodels.ViewModelFactory

class LoginFragment : Fragment() {

    private var _binding: FragmentLoginBinding? = null
    private val binding get() = _binding!!
    
    private lateinit var authViewModel: AuthViewModel
    private lateinit var sessionManager: SessionManager

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View {
        _binding = FragmentLoginBinding.inflate(inflater, container, false)
        return binding.root
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
        // Initialize session manager
        sessionManager = SessionManager(requireContext())
        
        // Check if user is already logged in
        if (sessionManager.isLoggedIn()) {
            navigateToHome()
            return
        }
        
        // Initialize ViewModel
        val database = AppDatabase.getDatabase(requireContext())
        val userRepository = UserRepository(database.userDao())
        val factory = ViewModelFactory(userRepository = userRepository)
        authViewModel = ViewModelProvider(this, factory)[AuthViewModel::class.java]
        
        // Setup login button
        binding.btnLogin.setOnClickListener {
            login()
        }
        
        // Setup register link
        binding.tvRegisterLink.setOnClickListener {
            findNavController().navigate(R.id.action_loginFragment_to_registerFragment)
        }
        
        // Observe login result
        authViewModel.loginResult.observe(viewLifecycleOwner) { result ->
            result.onSuccess { user ->
                sessionManager.saveUserSession(user.userId, user.username)
                navigateToHome()
            }.onFailure { exception ->
                Toast.makeText(requireContext(), exception.message, Toast.LENGTH_SHORT).show()
            }
        }
    }
    
    private fun login() {
        val username = binding.etUsername.text.toString().trim()
        val password = binding.etPassword.text.toString().trim()
        
        if (username.isEmpty() || password.isEmpty()) {
            Toast.makeText(requireContext(), "Please fill all fields", Toast.LENGTH_SHORT).show()
            return
        }
        
        authViewModel.login(username, password)
    }
    
    private fun navigateToHome() {
        findNavController().navigate(R.id.action_loginFragment_to_homeFragment)
    }

    override fun onDestroyView() {
        super.onDestroyView()
        _binding = null
    }
}
```

#### RegisterFragment.kt
```kotlin
package com.example.budgettrackerapp.ui.auth

import android.os.Bundle
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.Toast
import androidx.fragment.app.Fragment
import androidx.lifecycle.ViewModelProvider
import androidx.navigation.fragment.findNavController
import com.example.budgettrackerapp.R
import com.example.budgettrackerapp.data.AppDatabase
import com.example.budgettrackerapp.databinding.FragmentRegisterBinding
import com.example.budgettrackerapp.repository.UserRepository
import com.example.budgettrackerapp.viewmodels.AuthViewModel
import com.example.budgettrackerapp.viewmodels.ViewModelFactory

class RegisterFragment : Fragment() {

    private var _binding: FragmentRegisterBinding? = null
    private val binding get() = _binding!!
    
    private lateinit var authViewModel: AuthViewModel

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View {
        _binding = FragmentRegisterBinding.inflate(inflater, container, false)
        return binding.root
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
        // Initialize ViewModel
        val database = AppDatabase.getDatabase(requireContext())
        val userRepository = UserRepository(database.userDao())
        val factory = ViewModelFactory(userRepository = userRepository)
        authViewModel = ViewModelProvider(this, factory)[AuthViewModel::class.java]
        
        // Setup register button
        binding.btnRegister.setOnClickListener {
            register()
        }
        
        // Setup login link
        binding.tvLoginLink.setOnClickListener {
            findNavController().navigate(R.id.action_registerFragment_to_loginFragment)
        }
        
        // Observe register result
        authViewModel.registerResult.observe(viewLifecycleOwner) { result ->
            result.onSuccess {
                Toast.makeText(requireContext(), "Registration successful! Please login.", Toast.LENGTH_SHORT).show()
                findNavController().navigate(R.id.action_registerFragment_to_loginFragment)
            }.onFailure { exception ->
                Toast.makeText(requireContext(), exception.message, Toast.LENGTH_SHORT).show()
            }
        }
    }
    
    private fun register() {
        val username = binding.etUsername.text.toString().trim()
        val password = binding.etPassword.text.toString().trim()
        val confirmPassword = binding.etConfirmPassword.text.toString().trim()
        
        if (username.isEmpty() || password.isEmpty() || confirmPassword.isEmpty()) {
            Toast.makeText(requireContext(), "Please fill all fields", Toast.LENGTH_SHORT).show()
            return
        }
        
        if (password != confirmPassword) {
            Toast.makeText(requireContext(), "Passwords do not match", Toast.LENGTH_SHORT).show()
            return
        }
        
        authViewModel.register(username, password)
    }

    override fun onDestroyView() {
        super.onDestroyView()
        _binding = null
    }
}
```

## Implementing Camera Functionality

### 1. Create a utility class for camera and file operations

#### FileUtils.kt
```kotlin
package com.example.budgettrackerapp.utils

import android.content.Context
import android.graphics.Bitmap
import android.net.Uri
import android.os.Environment
import androidx.core.content.FileProvider
import java.io.File
import java.io.FileOutputStream
import java.io.IOException
import java.text.SimpleDateFormat
import java.util.Date
import java.util.Locale

object FileUtils {
    
    fun createImageFile(context: Context): File {
        // Create an image file name
        val timeStamp = SimpleDateFormat("yyyyMMdd_HHmmss", Locale.getDefault()).format(Date())
        val storageDir = context.getExternalFilesDir(Environment.DIRECTORY_PICTURES)
        
        return File.createTempFile(
            "JPEG_${timeStamp}_", /* prefix */
            ".jpg", /* suffix */
            storageDir /* directory */
        )
    }
    
    fun getUriForFile(context: Context, file: File): Uri {
        return FileProvider.getUriForFile(
            context,
            "${context.packageName}.fileprovider",
            file
        )
    }
    
    fun saveBitmapToFile(bitmap: Bitmap, file: File): Boolean {
        return try {
            FileOutputStream(file).use { out ->
                bitmap.compress(Bitmap.CompressFormat.JPEG, 90, out)
            }
            true
        } catch (e: IOException) {
            e.printStackTrace()
            false
        }
    }
}
```

### 2. Add File Provider configuration

Create a new XML file in `res/xml/file_paths.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<paths>
    <external-files-path
        name="my_images"
        path="Pictures" />
</paths>
```

Add the FileProvider to your AndroidManifest.xml within the application tag:

```xml
<provider
    android:name="androidx.core.content.FileProvider"
    android:authorities="${applicationId}.fileprovider"
    android:exported="false"
    android:grantUriPermissions="true">
    <meta-data
        android:name="android.support.FILE_PROVIDER_PATHS"
        android:resource="@xml/file_paths" />
</provider>
```

### 3. Add necessary permissions to AndroidManifest.xml

```xml
<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" 
    android:maxSdkVersion="28" />
<uses-feature android:name="android.hardware.camera" />
```

## Putting It All Together

### 1. Create a custom Application class

#### BudgetTrackerApp.kt
```kotlin
package com.example.budgettrackerapp

import android.app.Application
import com.example.budgettrackerapp.data.AppDatabase
import com.example.budgettrackerapp.repository.BudgetGoalRepository
import com.example.budgettrackerapp.repository.CategoryRepository
import com.example.budgettrackerapp.repository.ExpenseRepository
import com.example.budgettrackerapp.repository.UserRepository

class BudgetTrackerApp : Application() {
    
    // Lazy database instance
    val database by lazy { AppDatabase.getDatabase(this) }
    
    // Repositories
    val userRepository by lazy { UserRepository(database.userDao()) }
    val categoryRepository by lazy { CategoryRepository(database.categoryDao()) }
    val expenseRepository by lazy { ExpenseRepository(database.expenseDao()) }
    val budgetGoalRepository by lazy { BudgetGoalRepository(database.budgetGoalDao()) }
}
```

### 2. Update AndroidManifest.xml to use the Application class

```xml
<application
    android:name=".BudgetTrackerApp"
    android:allowBackup="true"
    android:icon="@mipmap/ic_launcher"
    android:label="@string/app_name"
    android:roundIcon="@mipmap/ic_launcher_round"
    android:supportsRtl="true"
    android:theme="@style/Theme.BudgetTrackerApp">
    <!-- Rest of the manifest -->
</application>
```

### 3. Create Adapters for RecyclerViews

#### CategoryAdapter.kt
```kotlin
package com.example.budgettrackerapp.ui.adapters

import android.view.LayoutInflater
import android.view.ViewGroup
import androidx.recyclerview.widget.DiffUtil
import androidx.recyclerview.widget.ListAdapter
import androidx.recyclerview.widget.RecyclerView
import com.example.budgettrackerapp.data.entities.Category
import com.example.budgettrackerapp.databinding.ItemCategoryBinding

class CategoryAdapter(
    private val onCategoryClick: (Category) -> Unit,
    private val onEditClick: (Category) -> Unit,
    private val onDeleteClick: (Category) -> Unit
) : ListAdapter<Category, CategoryAdapter.CategoryViewHolder>(CategoryDiffCallback()) {

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): CategoryViewHolder {
        val binding = ItemCategoryBinding.inflate(
            LayoutInflater.from(parent.context),
            parent,
            false
        )
        return CategoryViewHolder(binding)
    }

    override fun onBindViewHolder(holder: CategoryViewHolder, position: Int) {
        val category = getItem(position)
        holder.bind(category)
    }

    inner class CategoryViewHolder(private val binding: ItemCategoryBinding) :
        RecyclerView.ViewHolder(binding.root) {

        init {
            binding.root.setOnClickListener {
                val position = bindingAdapterPosition
                if (position != RecyclerView.NO_POSITION) {
                    onCategoryClick(getItem(position))
                }
            }

            binding.btnEdit.setOnClickListener {
                val position = bindingAdapterPosition
                if (position != RecyclerView.NO_POSITION) {
                    onEditClick(getItem(position))
                }
            }

            binding.btnDelete.setOnClickListener {
                val position = bindingAdapterPosition
                if (position != RecyclerView.NO_POSITION) {
                    onDeleteClick(getItem(position))
                }
            }
        }

        fun bind(category: Category) {
            binding.tvCategoryName.text = category.name
        }
    }

    private class CategoryDiffCallback : DiffUtil.ItemCallback<Category>() {
        override fun areItemsTheSame(oldItem: Category, newItem: Category): Boolean {
            return oldItem.categoryId == newItem.categoryId
        }

        override fun areContentsTheSame(oldItem: Category, newItem: Category): Boolean {
            return oldItem == newItem
        }
    }
}
```

#### ExpenseAdapter.kt
```kotlin
package com.example.budgettrackerapp.ui.adapters

import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import androidx.recyclerview.widget.DiffUtil
import androidx.recyclerview.widget.ListAdapter
import androidx.recyclerview.widget.RecyclerView
import com.bumptech.glide.Glide
import com.example.budgettrackerapp.data.entities.Expense
import com.example.budgettrackerapp.databinding.ItemExpenseBinding
import java.text.SimpleDateFormat
import java.util.Date
import java.util.Locale

class ExpenseAdapter(
    private val categoryMap: Map<Long, String>,
    private val onExpenseClick: (Expense) -> Unit
) : ListAdapter<Expense, ExpenseAdapter.ExpenseViewHolder>(ExpenseDiffCallback()) {

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ExpenseViewHolder {
        val binding = ItemExpenseBinding.inflate(
            LayoutInflater.from(parent.context),
            parent,
            false
        )
        return ExpenseViewHolder(binding)
    }

    override fun onBindViewHolder(holder: ExpenseViewHolder, position: Int) {
        val expense = getItem(position)
        holder.bind(expense)
    }

    inner class ExpenseViewHolder(private val binding: ItemExpenseBinding) :
        RecyclerView.ViewHolder(binding.root) {

        init {
            binding.root.setOnClickListener {
                val position = bindingAdapterPosition
                if (position != RecyclerView.NO_POSITION) {
                    onExpenseClick(getItem(position))
                }
            }
        }

        fun bind(expense: Expense) {
            val dateFormat = SimpleDateFormat("MMM dd, yyyy", Locale.getDefault())
            
            binding.tvExpenseAmount.text = "${expense.amount}"
            binding.tvExpenseDescription.text = expense.description
            binding.tvExpenseDate.text = dateFormat.format(Date(expense.date))
            binding.tvExpenseCategory.text = categoryMap[expense.categoryId] ?: "Unknown"
            
            if (expense.photoPath != null) {
                binding.ivExpensePhotoIndicator.visibility = View.VISIBLE
            } else {
                binding.ivExpensePhotoIndicator.visibility = View.GONE
            }
        }
    }

    private class ExpenseDiffCallback : DiffUtil.ItemCallback<Expense>() {
        override fun areItemsTheSame(oldItem: Expense, newItem: Expense): Boolean {
            return oldItem.expenseId == newItem.expenseId
        }

        override fun areContentsTheSame(oldItem: Expense, newItem: Expense): Boolean {
            return oldItem == newItem
        }
    }
}
```

### 4. Create Custom ViewModelFactory Extension

#### ViewModelFactoryExt.kt
```kotlin
package com.example.budgettrackerapp.utils

import androidx.fragment.app.Fragment
import androidx.lifecycle.ViewModel
import androidx.lifecycle.ViewModelProvider
import com.example.budgettrackerapp.BudgetTrackerApp

/**
 * Extension function to create ViewModels with app repositories
 */
inline fun <reified T : ViewModel> Fragment.getViewModel(): T {
    val application = requireActivity().application as BudgetTrackerApp
    
    return ViewModelProvider(this, ViewModelProvider.Factory {
        when {
            T::class.java.isAssignableFrom(com.example.budgettrackerapp.viewmodels.AuthViewModel::class.java) -> {
                com.example.budgettrackerapp.viewmodels.AuthViewModel(application.userRepository) as T
            }
            T::class.java.isAssignableFrom(com.example.budgettrackerapp.viewmodels.CategoryViewModel::class.java) -> {
                com.example.budgettrackerapp.viewmodels.CategoryViewModel(application.categoryRepository) as T
            }
            T::class.java.isAssignableFrom(com.example.budgettrackerapp.viewmodels.ExpenseViewModel::class.java) -> {
                com.example.budgettrackerapp.viewmodels.ExpenseViewModel(application.expenseRepository) as T
            }
            T::class.java.isAssignableFrom(com.example.budgettrackerapp.viewmodels.BudgetGoalViewModel::class.java) -> {
                com.example.budgettrackerapp.viewmodels.BudgetGoalViewModel(application.budgetGoalRepository) as T
            }
            else -> throw IllegalArgumentException("Unknown ViewModel class: ${T::class.java.name}")
        }
    })[T::class.java]
}
```

## Final Steps and Testing

### 1. Complete the implementation of all fragments

Implement the remaining fragment classes (HomeFragment, CategoriesFragment, etc.) following the patterns shown in the LoginFragment and RegisterFragment samples.

### 2. Create bottom_nav_menu.xml in res/menu

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:id="@+id/homeFragment"
        android:icon="@android:drawable/ic_menu_compass"
        android:title="Home" />
    <item
        android:id="@+id/transactionsFragment"
        android:icon="@android:drawable/ic_menu_recent_history"
        android:title="Transactions" />
    <item
        android:id="@+id/dashboardFragment"
        android:icon="@android:drawable/ic_menu_gallery"
        android:title="Dashboard" />
</menu>
```

### 3. Test the application

1. Run the application in an emulator or physical device
2. Test user registration and login functionality
3. Test adding categories
4. Test adding expenses with photos
5. Test setting budget goals
6. Test viewing transactions and dashboard stats

## Conclusion

This guide has provided a complete implementation for the budget tracker app with all the required features:

- User authentication (login/signup)
- Category management
- Expense entry with photo capability
- Budget goal setting
- Transaction listing with date filtering
- Expense analysis by category

The app follows MVVM architecture with Room database for local persistence, making it maintainable and scalable. The UI is designed to be user-friendly with intuitive navigation and clear presentation of data.