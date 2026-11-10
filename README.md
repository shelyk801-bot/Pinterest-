// ==========================================
// ALL-IN-ONE PROJECT CODE STRUCTURE
// ==========================================

// --- FILE 1: backend/requirements.txt ---
fastapi==0.110.0
uvicorn==0.28.0
google-genai==0.1.1
requests==2.31.0
pydantic==2.6.4
apscheduler==3.10.4
firebase-admin==6.5.0
python-dotenv==1.0.1
pillow==10.2.0

// --- FILE 2: backend/main.py ---
import os
import uvicorn
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from typing import Optional
from services.gemini_service import GeminiEngine
from services.pinterest_service import PinterestManager

app = FastAPI(title="Gemini Pinterest Creator API", version="1.0.0")

gemini_engine = GeminiEngine()
pinterest_mgr = PinterestManager()

class ProductImportRequest(BaseModel):
    url: str
    marketplace: Optional[str] = None

class ContentGenRequest(BaseModel):
    title: str
    description: str
    category: str

class PinterestPublishRequest(BaseModel):
    board_id: str
    image_url: str
    title: str
    description: str
    link: str
    access_token: str

@app.get("/")
def read_root():
    return {"status": "Active", "system": "Gemini Affiliate Engine Core"}

@app.post("/api/import-product")
async def import_product(req: ProductImportRequest):
    if "amazon" in req.url.lower():
        market = "Amazon"
    elif "aliexpress" in req.url.lower():
        market = "AliExpress"
    else:
        market = "Generic"
    return {"status": "success", "marketplace": market, "product_name": "Sample Affiliate Item", "original_url": req.url, "image_url": "https://via.placeholder.com/600x800", "price": "$29.99"}

@app.post("/api/generate-seo")
async def generate_seo(req: ContentGenRequest, api_key: str):
    result = await gemini_engine.generate_pinterest_seo(api_key=api_key, title=req.title, desc=req.description, category=req.category)
    return result

@app.post("/api/publish-pin")
async def publish_pin(req: PinterestPublishRequest):
    res = pinterest_mgr.create_pin(access_token=req.access_token, board_id=req.board_id, title=req.title, description=req.description, link=req.link, image_url=req.image_url)
    return res

if __name__ == "__main__":
    uvicorn.run("main:app", host="0.0.0.0", port=8000, reload=True)

// --- FILE 3: android/app/build.gradle.kts ---
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.android)
}

android {
    namespace = "com.example.pinterestcreator"
    compileSdk = 35

    defaultConfig {
        applicationId = "com.example.pinterestcreator"
        minSdk = 26
        targetSdk = 35
        versionCode = 1
        versionName = "1.0.0"
        testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"
    }
    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_17
        targetCompatibility = JavaVersion.VERSION_17
    }
    kotlinOptions {
        jvmTarget = "17"
    }
    buildFeatures {
        compose = true
    }
}

dependencies {
    implementation("androidx.core:core-ktx:1.12.0")
    implementation("androidx.lifecycle:lifecycle-runtime-ktx:2.7.0")
    implementation("androidx.activity:activity-compose:1.8.2")
    implementation(platform("androidx.compose:compose-bom:2024.02.01"))
    implementation("androidx.compose.ui:ui")
    implementation("androidx.compose.ui:ui-material3")
    implementation("io.coil-kt:coil-compose:2.6.0")
    implementation("com.squareup.retrofit2:retrofit:2.9.0")
    implementation("com.squareup.retrofit2:converter-gson:2.9.0")
}

// --- FILE 4: android/app/src/main/java/com/example/pinterestcreator/MainActivity.kt ---
package com.example.pinterestcreator

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.foundation.layout.*
import androidx.compose.material3.*
import androidx.compose.ui.Modifier
import androidx.compose.ui.graphics.Color
import com.example.pinterestcreator.ui.screens.DashboardScreen

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            MaterialTheme {
                Surface(modifier = Modifier.fillMaxSize()) {
                    DashboardScreen()
                }
            }
        }
    }
}

// --- FILE 5: .github/workflows/android-build.yml ---
name: Build Android Debug APK
on:
  push:
    branches: [ "main" ]
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout Source Code
      uses: actions/checkout@v4
    - name: Set up JDK 17
      uses: actions/setup-java@v4
      with:
        java-version: '17'
        distribution: 'temurin'
    - name: Grant Execute Permission for Gradle Wrapper
      run: chmod +x android/gradlew
    - name: Build Android Debug APK
      run: |
        cd android
        ./gradlew assembleDebug --stacktrace
    - name: Upload Debug APK Artifact
      uses: actions/upload-artifact@v4
      with:
        name: Gemini-Pinterest-Creator-Debug-APK
        path: android/app/build/outputs/apk/debug/app-debug.apk
