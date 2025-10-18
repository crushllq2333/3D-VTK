<template>
  <div class="paraview-style-container">
    <!-- 顶部工具栏 -->
    <div class="toolbar">
      <div class="toolbar-group">
        <button class="toolbar-btn" @click="openFile">
          <span class="icon">📁</span> 打开
        </button>
        <button class="toolbar-btn" @click="saveFile">
          <span class="icon">💾</span> 保存
        </button>
        <button class="toolbar-btn" @click="screenshot">
          <span class="icon">📷</span> 截图
        </button>
      </div>
      
      <div class="toolbar-group">
        <button class="toolbar-btn" @click="applyChanges">
          <span class="icon">⏯️</span> 应用
        </button>
        <button class="toolbar-btn" @click="resetView">
          <span class="icon">🔄</span> 重置
        </button>
        <button class="toolbar-btn" @click="deleteSelected">
          <span class="icon">❌</span> 删除
        </button>
      </div>
      
      <div class="toolbar-group">
        <span class="toolbar-label">显示模式:</span>
        <select class="toolbar-select" v-model="displayMode" @change="onDisplayModeChange">
          <option value="volume">体渲染</option>
          <option value="slice">切片</option>
          <option value="isosurface">等值面</option>
          <option value="scalarRange">标量范围</option>
        </select>
      </div>
      
      <div class="toolbar-group">
        <span class="toolbar-label">颜色映射:</span>
        <select class="toolbar-select" v-model="colorMap" @change="onColorMapChange">
          <option value="coolToWarm">冷到暖</option>
          <option value="rainbow">彩虹</option>
          <option value="grayscale">灰度</option>
          <option value="hot">热图</option>
        </select>
      </div>
      
      <div class="toolbar-group">
        <span class="toolbar-label">不透明度:</span>
        <input type="range" min="0" max="100" v-model="opacity" class="toolbar-input" @input="onOpacityChange">
        <span class="toolbar-value">{{ opacity }}%</span>
      </div>
      
      <div class="toolbar-group">
        <button class="toolbar-btn" @click="toggleRenderControls" :class="{ active: showRenderControls }">
          <span class="icon">⚙️</span> 渲染控制
        </button>
      </div>

      <div class="toolbar-group" style="margin-left: auto;">
        <button class="toolbar-btn" @click="toggleFullscreen">
          <span class="icon">⛶</span> 全屏
        </button>
        <button class="toolbar-btn language-btn" @click="toggleLanguage">
          <span class="icon">{{ currentLanguage === 'zh' ? '🇨🇳' : '🇺🇸' }}</span>
          {{ currentLanguage === 'zh' ? 'EN' : '中文' }}
        </button>
      </div>
    </div>
    
    <!-- 主内容区域 -->
    <div class="main-content">
      <!-- 左侧面板 -->
      <div class="left-panel" v-show="showLeftPanel">
        <!-- 渲染控制面板 -->
        <div class="panel-section" v-show="showRenderControls">
          <div class="panel-header">
            渲染控制
          </div>
          <div class="panel-content">
            <!-- 等值面控制 -->
            <div class="control-group" v-if="displayMode === 'isosurface'">
              <h4 class="control-title">等值面控制</h4>
              <div class="property-row">
                <label class="property-label">等值面值:</label>
                <input 
                  type="number" 
                  class="property-input" 
                  v-model.number="renderControls.isosurfaceValue"
                  :min="dataRange[0]" 
                  :max="dataRange[1]"
                  :step="(dataRange[1] - dataRange[0]) / 100"
                  @change="applyIsoSurface"
                >
              </div>
              <div class="property-row">
                <label class="property-label">表面不透明度:</label>
                <input 
                  type="range" 
                  min="0" 
                  max="100" 
                  v-model.number="renderControls.surfaceOpacity"
                  class="property-slider"
                  @input="applyIsoSurface"
                >
                <span class="property-value">{{ renderControls.surfaceOpacity }}%</span>
              </div>
              <div class="property-row">
                <button class="apply-btn" @click="applyIsoSurface">
                  应用等值面
                </button>
              </div>
            </div>

            <!-- 切面控制 -->
            <div class="control-group" v-if="displayMode === 'slice'">
              <h4 class="control-title">切面控制</h4>
              <div class="property-row">
                <label class="property-label">X 位置:</label>
                <input 
                  type="number" 
                  class="property-input" 
                  v-model.number="renderControls.sliceX"
                  :min="0" 
                  :max="dimensions[0] - 1"
                  @change="applySlice"
                >
                <input 
                  type="range" 
                  :min="0" 
                  :max="dimensions[0] - 1" 
                  v-model.number="renderControls.sliceX"
                  class="property-slider"
                  @input="applySlice"
                >
              </div>
              <div class="property-row">
                <label class="property-label">Y 位置:</label>
                <input 
                  type="number" 
                  class="property-input" 
                  v-model.number="renderControls.sliceY"
                  :min="0" 
                  :max="dimensions[1] - 1"
                  @change="applySlice"
                >
                <input 
                  type="range" 
                  :min="0" 
                  :max="dimensions[1] - 1" 
                  v-model.number="renderControls.sliceY"
                  class="property-slider"
                  @input="applySlice"
                >
              </div>
              <div class="property-row">
                <label class="property-label">Z 位置:</label>
                <input 
                  type="number" 
                  class="property-input" 
                  v-model.number="renderControls.sliceZ"
                  :min="0" 
                  :max="dimensions[2] - 1"
                  @change="applySlice"
                >
                <input 
                  type="range" 
                  :min="0" 
                  :max="dimensions[2] - 1" 
                  v-model.number="renderControls.sliceZ"
                  class="property-slider"
                  @input="applySlice"
                >
              </div>
              <div class="property-row">
                <button class="apply-btn" @click="applySlice">
                  应用切面
                </button>
              </div>
            </div>

            <!-- 标量值过滤控制 -->
            <div class="control-group" v-if="displayMode === 'scalarRange'">
              <h4 class="control-title">标量范围过滤</h4>
              <div class="property-row">
                <label class="property-label">最小值:</label>
                <input 
                  type="number" 
                  class="property-input" 
                  v-model.number="renderControls.scalarMin"
                  :min="dataRange[0]" 
                  :max="dataRange[1]"
                  :step="(dataRange[1] - dataRange[0]) / 100"
                  @change="applyScalarRange"
                >
                <input 
                  type="range" 
                  :min="dataRange[0]" 
                  :max="dataRange[1]" 
                  v-model.number="renderControls.scalarMin"
                  class="property-slider"
                  @input="applyScalarRange"
                >
              </div>
              <div class="property-row">
                <label class="property-label">最大值:</label>
                <input 
                  type="number" 
                  class="property-input" 
                  v-model.number="renderControls.scalarMax"
                  :min="dataRange[0]" 
                  :max="dataRange[1]"
                  :step="(dataRange[1] - dataRange[0]) / 100"
                  @change="applyScalarRange"
                >
                <input 
                  type="range" 
                  :min="dataRange[0]" 
                  :max="dataRange[1]" 
                  v-model.number="renderControls.scalarMax"
                  class="property-slider"
                  @input="applyScalarRange"
                >
              </div>
              <div class="property-row">
                <button class="apply-btn" @click="applyScalarRange">
                  应用标量范围
                </button>
              </div>
            </div>

            <!-- 通用控制 -->
            <div class="control-group">
              <h4 class="control-title">通用信息</h4>
              <div class="property-row">
                <label class="property-label">数据范围:</label>
                <span class="property-value">{{ dataRange[0].toFixed(2) }} - {{ dataRange[1].toFixed(2) }}</span>
              </div>
              <div class="property-row">
                <label class="property-label">维度:</label>
                <span class="property-value">{{ dimensions.join(' × ') }}</span>
              </div>
            </div>
          </div>
        </div>

        <!-- 属性面板 -->
        <div class="panel-section">
          <div class="panel-header">
            属性
          </div>
          <div class="panel-content">
            <div class="property-row">
              <input type="checkbox" id="skip-zero" v-model="properties.skipZeroTime" class="property-checkbox" @change="onPropertyChange">
              <label for="skip-zero" class="property-label">跳过零时间</label>
            </div>
            
            <div class="property-row">
              <label class="property-label">案例类型</label>
              <select class="property-input" v-model="properties.caseType" @change="onPropertyChange">
                <option value="steady">稳态</option>
                <option value="transient">瞬态</option>
              </select>
            </div>
            
            <div class="property-row">
              <label class="property-label">标签大小</label>
              <select class="property-input" v-model="properties.labelSize" @change="onPropertyChange">
                <option value="32">32-bit</option>
                <option value="64">64-bit</option>
              </select>
            </div>
          </div>
        </div>
      </div>
      
      <!-- 渲染区域 -->
      <div class="render-area">
        <div ref="container" class="vtk-container"></div>
        <div class="view-controls">
          <button class="control-btn" @click="rotateView('left')" title="Rotate Left">↶</button>
          <button class="control-btn" @click="rotateView('right')" title="Rotate Right">↷</button>
          <button class="control-btn" @click="resetCamera" title="Reset Camera">⨀</button>
          <button class="control-btn" @click="zoomIn" title="Zoom In">+</button>
          <button class="control-btn" @click="zoomOut" title="Zoom Out">-</button>
          <button class="panel-toggle" @click="showLeftPanel = !showLeftPanel">
            {{ showLeftPanel ? '◀' : '▶' }}
          </button>
        </div>
        
        <!-- 加载指示器 -->
        <div v-if="isLoading" class="loading-overlay">
          <div class="loading-spinner"></div>
          <p>加载中...</p>
        </div>
      </div>
    </div>
    
    <!-- 底部状态栏 -->
    <div class="status-bar">
      <div>{{ statusMessage }}</div>
      <div>VTK.js Volume Renderer | 显示模式: {{ getDisplayModeText() }}</div>
    </div>

    <!-- 消息提示 -->
    <div v-if="showMessage" class="message-toast" :class="messageType">
      {{ messageText }}
    </div>
  </div>
</template>

<script setup>
import { onMounted, ref, reactive, computed } from 'vue';

// VTK.js imports
import '@kitware/vtk.js/favicon';
import '@kitware/vtk.js/Rendering/Profiles/Volume';
import vtkFullScreenRenderWindow from '@kitware/vtk.js/Rendering/Misc/FullScreenRenderWindow';
import vtkVolume from '@kitware/vtk.js/Rendering/Core/Volume';
import vtkVolumeMapper from '@kitware/vtk.js/Rendering/Core/VolumeMapper';
import vtkColorTransferFunction from '@kitware/vtk.js/Rendering/Core/ColorTransferFunction';
import vtkPiecewiseFunction from '@kitware/vtk.js/Common/DataModel/PiecewiseFunction';
import vtkImageData from '@kitware/vtk.js/Common/DataModel/ImageData';
import vtkDataArray from '@kitware/vtk.js/Common/Core/DataArray';
import vtkBoundingBox from '@kitware/vtk.js/Common/DataModel/BoundingBox';

// 切面渲染相关导入
import vtkImageMapper from '@kitware/vtk.js/Rendering/Core/ImageMapper';
import vtkImageSlice from '@kitware/vtk.js/Rendering/Core/ImageSlice';

// DOM 引用
const container = ref(null);

// 响应式数据
const showLeftPanel = ref(true);
const displayMode = ref('volume');
const colorMap = ref('coolToWarm');
const opacity = ref(40);
const statusMessage = ref('就绪');
const isLoading = ref(false);
const showRenderControls = ref(true);
const showMessage = ref(false);
const messageText = ref('');
const messageType = ref('info');
const currentLanguage = ref('zh');

// 新增数据
const dataRange = ref([0, 1]);
const dimensions = ref([100, 100, 100]);

// 渲染控制
const renderControls = reactive({
  isosurfaceValue: 0.5,
  surfaceOpacity: 80,
  sliceX: 50,
  sliceY: 50,
  sliceZ: 50,
  scalarMin: 0,
  scalarMax: 1
});

// 属性配置
const properties = reactive({
  skipZeroTime: true,
  caseType: 'steady',
  labelSize: '32'
});

// VTK 相关变量
let fullScreenRenderer = null;
let renderer = null;
let renderWindow = null;
let volumeActor = null;
let volumeMapper = null;
let colorTransferFunction = null;
let opacityFunction = null;
let imageData = null;

// 新增切面相关变量
let sliceActors = {
  x: null,
  y: null, 
  z: null
};
let sliceMappers = {
  x: null,
  y: null,
  z: null
};

// 方法
const showToast = (text, type = 'info') => {
  messageText.value = text;
  messageType.value = type;
  showMessage.value = true;
  setTimeout(() => {
    showMessage.value = false;
  }, 3000);
};

const toggleLanguage = () => {
  currentLanguage.value = currentLanguage.value === 'zh' ? 'en' : 'zh';
  statusMessage.value = '就绪';
};

const toggleFullscreen = () => {
  if (!document.fullscreenElement) {
    document.documentElement.requestFullscreen().catch(err => {
      console.error(`全屏请求错误: ${err.message}`);
    });
  } else {
    document.exitFullscreen();
  }
};

const openFile = () => {
  showToast('打开 就绪', 'info');
};

const saveFile = () => {
  showToast('保存 就绪', 'info');
};

const screenshot = () => {
  showToast('截图 就绪', 'success');
};

const applyChanges = () => {
  showToast('应用 - 更改已应用', 'success');
};

const resetView = () => {
  if (renderer) {
    renderer.resetCamera();
    renderWindow.render();
  }
  showToast('重置 - 视图已重置', 'info');
};

const deleteSelected = () => {
  showToast('删除 就绪', 'warning');
};

const toggleRenderControls = () => {
  showRenderControls.value = !showRenderControls.value;
  showToast(`渲染控制 ${showRenderControls.value ? '显示' : '隐藏'}`, 'info');
};

const applyIsoSurface = () => {
  if (!opacityFunction || !volumeActor) return;
  
  // 使用不透明度函数来模拟等值面
  const isovalue = renderControls.isosurfaceValue;
  const surfaceOpacity = renderControls.surfaceOpacity / 100;
  
  opacityFunction.removeAllPoints();
  
  // 创建等值面效果：在等值面值附近设置高不透明度，其他地方低不透明度
  const range = dataRange.value[1] - dataRange.value[0];
  const transition = range * 0.02; // 2% 的过渡区域
  
  opacityFunction.addPoint(dataRange.value[0], 0.0);
  opacityFunction.addPoint(isovalue - transition, 0.0);
  opacityFunction.addPoint(isovalue, surfaceOpacity);
  opacityFunction.addPoint(isovalue + transition, 0.0);
  opacityFunction.addPoint(dataRange.value[1], 0.0);
  
  const prop = volumeActor.getProperty();
  prop.setScalarOpacity(0, opacityFunction);
  renderWindow.render();
  
  showToast(`等值面: ${isovalue.toFixed(2)}`, 'success');
};

const applySlice = () => {
  if (!imageData) return;
  
  // 清除现有的切面
  clearSlices();
  
  const [nx, ny, nz] = dimensions.value;
  
  // 创建 X 切面
  if (renderControls.sliceX >= 0 && renderControls.sliceX < nx) {
    createSlice('x', renderControls.sliceX);
  }
  
  // 创建 Y 切面
  if (renderControls.sliceY >= 0 && renderControls.sliceY < ny) {
    createSlice('y', renderControls.sliceY);
  }
  
  // 创建 Z 切面
  if (renderControls.sliceZ >= 0 && renderControls.sliceZ < nz) {
    createSlice('z', renderControls.sliceZ);
  }
  
  renderWindow.render();
  showToast(`切面已应用: X=${renderControls.sliceX}, Y=${renderControls.sliceY}, Z=${renderControls.sliceZ}`, 'success');
};

// 创建切面的辅助方法
const createSlice = (axis, position) => {
  // 创建映射器
  const mapper = vtkImageMapper.newInstance();
  mapper.setInputData(imageData);
  
  // 根据轴设置切片模式
  switch (axis) {
    case 'x':
      mapper.setSlicingMode(vtkImageMapper.SlicingMode.I);
      mapper.setISlice(position);
      break;
    case 'y':
      mapper.setSlicingMode(vtkImageMapper.SlicingMode.J);
      mapper.setJSlice(position);
      break;
    case 'z':
      mapper.setSlicingMode(vtkImageMapper.SlicingMode.K);
      mapper.setKSlice(position);
      break;
  }
  
  // 创建切片 Actor
  const actor = vtkImageSlice.newInstance();
  actor.setMapper(mapper);
  
  // 设置颜色映射
  const property = actor.getProperty();
  property.setRGBTransferFunction(0, colorTransferFunction);
  
  // 设置不透明度
  property.setOpacity(0.8);
  
  // 添加到渲染器
  renderer.addActor(actor);
  
  // 保存引用
  sliceMappers[axis] = mapper;
  sliceActors[axis] = actor;
};

// 清除所有切面的方法
const clearSlices = () => {
  Object.values(sliceActors).forEach(actor => {
    if (actor) {
      renderer.removeActor(actor);
    }
  });
  
  sliceActors = { x: null, y: null, z: null };
  sliceMappers = { x: null, y: null, z: null };
};

const applyScalarRange = () => {
  if (!opacityFunction || !volumeActor) return;
  
  // 更新不透明度函数以过滤标量范围
  opacityFunction.removeAllPoints();
  
  const minVal = renderControls.scalarMin;
  const maxVal = renderControls.scalarMax;
  
  // 在范围边界设置不透明度过渡
  const range = dataRange.value[1] - dataRange.value[0];
  const transition = range * 0.01; // 1% 的过渡区域
  
  opacityFunction.addPoint(minVal - transition, 0.0);
  opacityFunction.addPoint(minVal, opacity.value / 100);
  opacityFunction.addPoint(maxVal, opacity.value / 100);
  opacityFunction.addPoint(maxVal + transition, 0.0);
  
  const prop = volumeActor.getProperty();
  prop.setScalarOpacity(0, opacityFunction);
  renderWindow.render();
  
  showToast(`标量范围: ${minVal.toFixed(2)} - ${maxVal.toFixed(2)}`, 'success');
};

const onDisplayModeChange = () => {
  showToast(`显示模式: ${getDisplayModeText()}`, 'info');
  
  // 根据显示模式切换渲染
  switch (displayMode.value) {
    case 'volume':
      showVolumeRendering();
      break;
    case 'slice':
      showSliceRendering();
      break;
    case 'isosurface':
      applyIsoSurface();
      break;
    case 'scalarRange':
      applyScalarRange();
      break;
  }
};

// 显示切面渲染的方法
const showSliceRendering = () => {
  // 清除体渲染
  if (volumeActor) {
    volumeActor.setVisibility(false);
  }
  
  // 应用切面
  applySlice();
};

// 显示体渲染的方法
const showVolumeRendering = () => {
  // 清除切面
  clearSlices();
  
  // 显示体渲染
  if (volumeActor) {
    volumeActor.setVisibility(true);
    
    // 重置不透明度函数为体渲染模式
    if (opacityFunction && volumeActor) {
      opacityFunction.removeAllPoints();
      opacityFunction.addPoint(dataRange.value[0], 0.0);
      opacityFunction.addPoint(dataRange.value[1], opacity.value / 100);
      const prop = volumeActor.getProperty();
      prop.setScalarOpacity(0, opacityFunction);
    }
  }
  
  renderWindow.render();
};

const onColorMapChange = () => {
  showToast(`颜色映射: ${getColorMapText()}`, 'info');
  updateColorMap();
};

const onOpacityChange = () => {
  if (opacityFunction && volumeActor) {
    const prop = volumeActor.getProperty();
    const range = opacityFunction.getRange();
    opacityFunction.removeAllPoints();
    opacityFunction.addPoint(range[0], 0.0);
    opacityFunction.addPoint(range[1], opacity.value / 100);
    prop.setScalarOpacity(0, opacityFunction);
    renderWindow.render();
  }
};

const onPropertyChange = () => {
  showToast('更改已应用', 'info');
};

const rotateView = (direction) => {
  if (renderer && renderer.getActiveCamera()) {
    const camera = renderer.getActiveCamera();
    const angle = direction === 'left' ? 15 : -15;
    camera.azimuth(angle);
    renderWindow.render();
  }
};

const resetCamera = () => {
  if (renderer) {
    renderer.resetCamera();
    renderWindow.render();
  }
};

const zoomIn = () => {
  if (renderer && renderer.getActiveCamera()) {
    const camera = renderer.getActiveCamera();
    camera.zoom(1.2);
    renderWindow.render();
  }
};

const zoomOut = () => {
  if (renderer && renderer.getActiveCamera()) {
    const camera = renderer.getActiveCamera();
    camera.zoom(0.8);
    renderWindow.render();
  }
};

const getDisplayModeText = () => {
  const modes = {
    volume: '体渲染',
    slice: '切片',
    isosurface: '等值面',
    scalarRange: '标量范围'
  };
  return modes[displayMode.value] || displayMode.value;
};

const getColorMapText = () => {
  const maps = {
    coolToWarm: '冷到暖',
    rainbow: '彩虹',
    grayscale: '灰度',
    hot: '热图'
  };
  return maps[colorMap.value] || colorMap.value;
};

const updateColorMap = () => {
  if (!colorTransferFunction || !volumeActor) return;

  colorTransferFunction.removeAllPoints();
  
  switch (colorMap.value) {
    case 'coolToWarm':
      colorTransferFunction.addRGBPoint(dataRange.value[0], 0.0, 0.0, 1.0);
      colorTransferFunction.addRGBPoint((dataRange.value[0] + dataRange.value[1]) / 2, 0.0, 1.0, 0.0);
      colorTransferFunction.addRGBPoint(dataRange.value[1], 1.0, 0.0, 0.0);
      break;
    case 'rainbow':
      colorTransferFunction.addRGBPoint(dataRange.value[0], 0.5, 0.0, 1.0);
      colorTransferFunction.addRGBPoint(dataRange.value[0] + (dataRange.value[1] - dataRange.value[0]) * 0.25, 0.0, 0.0, 1.0);
      colorTransferFunction.addRGBPoint(dataRange.value[0] + (dataRange.value[1] - dataRange.value[0]) * 0.5, 0.0, 1.0, 1.0);
      colorTransferFunction.addRGBPoint(dataRange.value[0] + (dataRange.value[1] - dataRange.value[0]) * 0.75, 0.0, 1.0, 0.0);
      colorTransferFunction.addRGBPoint(dataRange.value[1], 1.0, 1.0, 0.0);
      break;
    case 'grayscale':
      colorTransferFunction.addRGBPoint(dataRange.value[0], 0.0, 0.0, 0.0);
      colorTransferFunction.addRGBPoint(dataRange.value[1], 1.0, 1.0, 1.0);
      break;
    case 'hot':
      colorTransferFunction.addRGBPoint(dataRange.value[0], 0.0, 0.0, 0.0);
      colorTransferFunction.addRGBPoint(dataRange.value[0] + (dataRange.value[1] - dataRange.value[0]) * 0.33, 1.0, 0.0, 0.0);
      colorTransferFunction.addRGBPoint(dataRange.value[0] + (dataRange.value[1] - dataRange.value[0]) * 0.66, 1.0, 1.0, 0.0);
      colorTransferFunction.addRGBPoint(dataRange.value[1], 1.0, 1.0, 1.0);
      break;
  }

  const prop = volumeActor.getProperty();
  prop.setRGBTransferFunction(0, colorTransferFunction);
  renderWindow.render();
};

// VTK 初始化
onMounted(() => {
  if (!container.value) return;

  isLoading.value = true;
  statusMessage.value = '加载中...';

  const t0 = performance.now();
  
  // 创建全屏渲染窗口
  fullScreenRenderer = vtkFullScreenRenderWindow.newInstance({
    rootContainer: container.value,
    background: [0, 0, 0],
    useOffscreenBuffers: true,
  });

  renderer = fullScreenRenderer.getRenderer();
  renderWindow = fullScreenRenderer.getRenderWindow();

  // 加载 octree.bin 数据
  fetch('/octree.bin')
    .then(response => {
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      return response.arrayBuffer();
    })
    .then(arrayBuffer => {
      const t1 = performance.now();
      console.log(`📦 模型加载时间: ${(t1 - t0).toFixed(2)} ms`);
      statusMessage.value = `加载中... - ${(t1 - t0).toFixed(2)} ms`;

      const view = new DataView(arrayBuffer);
      let pos = 0;

      // 读取维度信息
      const nx = view.getUint32(pos, true); pos += 4;
      const ny = view.getUint32(pos, true); pos += 4;
      const nz = view.getUint32(pos, true); pos += 4;

      dimensions.value = [nx, ny, nz];
      const totalVoxels = nx * ny * nz;
      console.log(`🔵 体素数量: ${totalVoxels}`);
      statusMessage.value = `加载中... ${totalVoxels} voxels...`;

      const data = new Float32Array(totalVoxels);

      // 递归解析八叉树
      const parseNode = (minx, miny, minz, sx, sy, sz) => {
        const byte = view.getUint8(pos); pos++;
        if (byte === 0) {
          const value = view.getFloat32(pos, true); pos += 4;
          for (let dz = 0; dz < sz; dz++) {
            for (let dy = 0; dy < sy; dy++) {
              for (let dx = 0; dx < sx; dx++) {
                const x = minx + dx;
                const y = miny + dy;
                const z = minz + dz;
                const idx = z * ny * nx + y * nx + x;
                data[idx] = value;
              }
            }
          }
        } else {
          const halfx = Math.floor((sx + 1) / 2);
          const halfy = Math.floor((sy + 1) / 2);
          const halfz = Math.floor((sz + 1) / 2);
          for (let dz = 0; dz < 2; dz++) {
            for (let dy = 0; dy < 2; dy++) {
              for (let dx = 0; dx < 2; dx++) {
                const cx = minx + dx * halfx;
                const cy = miny + dy * halfy;
                const cz = minz + dz * halfz;
                const csx = dx === 0 ? halfx : sx - halfx;
                const csy = dy === 0 ? halfy : sy - halfy;
                const csz = dz === 0 ? halfz : sz - halfz;
                parseNode(cx, cy, cz, csx, csy, csz);
              }
            }
          }
        }
      };

      parseNode(0, 0, 0, nx, ny, nz);
      
      // 创建 VTK 图像数据
      imageData = vtkImageData.newInstance();
      imageData.setDimensions(nx, ny, nz);
      imageData.setOrigin(0, 0, 0);
      imageData.setSpacing(20, 20, 20);

      const scalars = vtkDataArray.newInstance({
        name: 'scalars',
        values: data,
        numberOfComponents: 1,
      });
      imageData.getPointData().setScalars(scalars);

      const range = scalars.getRange();
      dataRange.value = [range[0], range[1]];
      
      // 初始化渲染控制值
      renderControls.isosurfaceValue = (range[0] + range[1]) / 2;
      renderControls.sliceX = Math.floor(nx / 2);
      renderControls.sliceY = Math.floor(ny / 2);
      renderControls.sliceZ = Math.floor(nz / 2);
      renderControls.scalarMin = range[0];
      renderControls.scalarMax = range[1];

      // 创建体渲染
      volumeActor = vtkVolume.newInstance();
      volumeMapper = vtkVolumeMapper.newInstance();
      volumeMapper.setInputData(imageData);

      const sampleDistance = 0.7 * Math.sqrt(
        imageData.getSpacing().map(v => v * v).reduce((a, b) => a + b, 0)
      );
      volumeMapper.setSampleDistance(sampleDistance);
      volumeActor.setMapper(volumeMapper);

      // 颜色传输函数
      colorTransferFunction = vtkColorTransferFunction.newInstance();
      updateColorMap();

      // 不透明度函数
      opacityFunction = vtkPiecewiseFunction.newInstance();
      opacityFunction.addPoint(range[0], 0.0);
      opacityFunction.addPoint(range[1], opacity.value / 100);

      const prop = volumeActor.getProperty();
      prop.setRGBTransferFunction(0, colorTransferFunction);
      prop.setScalarOpacity(0, opacityFunction);
      prop.setScalarOpacityUnitDistance(
        0,
        vtkBoundingBox.getDiagonalLength(imageData.getBounds()) /
        Math.max(...imageData.getDimensions())
      );
      prop.setInterpolationTypeToLinear();
      prop.setGradientOpacityMinimumValue(0, 0);
      prop.setGradientOpacityMaximumValue(0, (range[1] - range[0]) * 0.05);
      prop.setAmbient(0.2);
      prop.setDiffuse(0.7);
      prop.setSpecular(0.3);
      prop.setSpecularPower(8.0);

      // 添加到渲染器
      renderer.addVolume(volumeActor);
      renderer.resetCamera();
      renderWindow.render();

      isLoading.value = false;
      const t2 = performance.now();
      console.log(`🖼️ 渲染完成: ${(t2 - t1).toFixed(2)} ms`);
      statusMessage.value = `就绪 - 总时间: ${(t2 - t0).toFixed(2)} ms`;
      showToast('就绪', 'success');
    })
    .catch(error => {
      console.error('❌ 加载数据失败:', error);
      statusMessage.value = `错误: ${error.message}`;
      isLoading.value = false;
      showToast(`加载失败: ${error.message}`, 'error');
    });
});
</script>



<style scoped>
.control-group {
  margin-bottom: 15px;
  padding-bottom: 10px;
  border-bottom: 1px solid #3e3e42;
}

.control-group:last-child {
  border-bottom: none;
}

.control-title {
  margin: 0 0 10px 0;
  font-size: 14px;
  color: #cccccc;
  font-weight: bold;
}

.property-value {
  font-size: 12px;
  color: #999;
  min-width: 40px;
  text-align: center;
}

.property-slider {
  flex: 1;
  margin: 0 8px;
}

.paraview-style-container {
  display: flex;
  flex-direction: column;
  height: 100vh;
  background-color: #1e1e1e;
  color: #d4d4d4;
  overflow: hidden;
}

/* 顶部工具栏样式 */
.toolbar {
  display: flex;
  background-color: #2d2d30;
  border-bottom: 1px solid #3e3e42;
  padding: 6px 10px;
  align-items: center;
  flex-wrap: wrap;
  gap: 10px;
}

.toolbar-group {
  display: flex;
  align-items: center;
  gap: 5px;
}

.toolbar-btn {
  background-color: #3c3c3c;
  border: 1px solid #5a5a5a;
  color: #d4d4d4;
  padding: 5px 10px;
  border-radius: 3px;
  cursor: pointer;
  font-size: 13px;
  display: flex;
  align-items: center;
  gap: 5px;
  white-space: nowrap;
  transition: all 0.2s ease;
}

.toolbar-btn:hover {
  background-color: #4a4a4a;
  transform: translateY(-1px);
}

.toolbar-btn.active {
  background-color: #007acc;
  border-color: #007acc;
}

.language-btn {
  background-color: #007acc;
  border-color: #007acc;
}

.language-btn:hover {
  background-color: #005a9e;
  border-color: #005a9e;
}

.toolbar-select {
  background-color: #3c3c3c;
  border: 1px solid #5a5a5a;
  color: #d4d4d4;
  padding: 5px 8px;
  border-radius: 3px;
  font-size: 13px;
  min-width: 120px;
  cursor: pointer;
}

.toolbar-input {
  background-color: #3c3c3c;
  border: 1px solid #5a5a5a;
  color: #d4d4d4;
  padding: 5px 8px;
  border-radius: 3px;
  font-size: 13px;
  width: 80px;
  cursor: pointer;
}

.toolbar-label {
  font-size: 13px;
  color: #cccccc;
  white-space: nowrap;
}

.toolbar-value {
  font-size: 13px;
  color: #cccccc;
  min-width: 40px;
}

/* 主内容区域 */
.main-content {
  display: flex;
  flex: 1;
  overflow: hidden;
}

/* 左侧面板 */
.left-panel {
  width: 300px;
  background-color: #252526;
  border-right: 1px solid #3e3e42;
  display: flex;
  flex-direction: column;
  overflow-y: auto;
  transition: width 0.3s ease;
}

.panel-section {
  margin: 10px;
  border: 1px solid #3e3e42;
  border-radius: 4px;
  overflow: hidden;
}

.panel-header {
  background-color: #2d2d30;
  padding: 8px 12px;
  font-weight: bold;
  font-size: 13px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.panel-content {
  padding: 10px;
}

.property-row {
  display: flex;
  align-items: center;
  margin-bottom: 8px;
  gap: 8px;
}

.property-label {
  flex: 1;
  font-size: 13px;
}

.property-input {
  background-color: #3c3c3c;
  border: 1px solid #5a5a5a;
  color: #d4d4d4;
  padding: 4px 8px;
  border-radius: 3px;
  font-size: 13px;
  width: 120px;
  cursor: pointer;
}

.property-checkbox {
  margin: 0;
  cursor: pointer;
}

.apply-btn {
  background-color: #007acc;
  border: 1px solid #007acc;
  color: white;
  padding: 8px 16px;
  border-radius: 3px;
  cursor: pointer;
  font-size: 13px;
  width: 100%;
  transition: background-color 0.2s ease;
}

.apply-btn:hover {
  background-color: #005a9e;
}

/* 渲染区域 */
.render-area {
  flex: 1;
  position: relative;
  background-color: #000;
}

.vtk-container {
  width: 100%;
  height: 100%;
}

.view-controls {
  position: absolute;
  top: 10px;
  right: 10px;
  display: flex;
  flex-direction: column;
  gap: 5px;
}

.control-btn {
  background-color: rgba(60, 60, 60, 0.8);
  border: 1px solid #5a5a5a;
  color: #d4d4d4;
  width: 32px;
  height: 32px;
  border-radius: 3px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 16px;
  transition: all 0.2s ease;
}

.control-btn:hover {
  background-color: rgba(74, 74, 74, 0.9);
  transform: scale(1.1);
}

.panel-toggle {
  position: absolute;
  top: 10px;
  left: 10px;
  background-color: rgba(60, 60, 60, 0.8);
  border: 1px solid #5a5a5a;
  color: #d4d4d4;
  width: 32px;
  height: 32px;
  border-radius: 3px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 10;
  transition: all 0.2s ease;
}

.panel-toggle:hover {
  background-color: rgba(74, 74, 74, 0.9);
  transform: scale(1.1);
}

/* 加载指示器 */
.loading-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.7);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  z-index: 100;
}

.loading-spinner {
  width: 40px;
  height: 40px;
  border: 4px solid #3c3c3c;
  border-top: 4px solid #007acc;
  border-radius: 50%;
  animation: spin 1s linear infinite;
  margin-bottom: 10px;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

/* 底部状态栏 */
.status-bar {
  background-color: #007acc;
  color: white;
  padding: 4px 10px;
  font-size: 12px;
  display: flex;
  justify-content: space-between;
}

/* 消息提示 */
.message-toast {
  position: fixed;
  top: 20px;
  right: 20px;
  padding: 12px 20px;
  border-radius: 4px;
  color: white;
  font-size: 14px;
  z-index: 1000;
  animation: slideIn 0.3s ease;
  max-width: 300px;
}

.message-toast.info {
  background-color: #007acc;
}

.message-toast.success {
  background-color: #107c10;
}

.message-toast.warning {
  background-color: #d83b01;
}

.message-toast.error {
  background-color: #e81123;
}

@keyframes slideIn {
  from {
    transform: translateX(100%);
    opacity: 0;
  }
  to {
    transform: translateX(0);
    opacity: 1;
  }
}

/* 响应式设计 */
@media (max-width: 1200px) {
  .left-panel {
    width: 250px;
  }
}

@media (max-width: 768px) {
  .toolbar {
    padding: 4px 6px;
  }
  
  .toolbar-btn {
    padding: 4px 6px;
    font-size: 12px;
  }
  
  .toolbar-select {
    min-width: 100px;
    font-size: 12px;
  }
  
  .left-panel {
    position: absolute;
    z-index: 50;
    height: 100%;
    box-shadow: 2px 0 10px rgba(0, 0, 0, 0.5);
  }
}

.icon {
  font-size: 14px;
}
</style>