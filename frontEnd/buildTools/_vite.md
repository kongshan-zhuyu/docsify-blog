# 自定义插件

## 全局自动导入组件-setupComponents

> 自动导入项目 `src/components` 目录下的所有组件，无需手动导入，直接在模板中使用即可。

使用方法：
```ts
import { setupComponents } from '@/plugins/setupComponents';

const app = createApp(App);
setupComponents(app);
```

> 插件实现：
```ts
import type { App, ComponentOptions } from 'vue';

/**
 * 将文件路径转换为大驼峰形式
 * @param path 文件路径
 * @returns 大驼峰形式的组件名
 */
const convertToCamelCase = (path: string): string => {
  // 提取文件名并移除.vue后缀（支持连字符）
  const fileName = path.replace(/^.+\/([\w-]+)\.vue$/, '$1');

  // 如果是index.vue文件，则使用父目录名称
  if (fileName === 'index') {
    // 提取父目录名称
    const parentDir = path.match(/^.+\/(\w+)\/index\.vue$/)?.[1] || 'index';
    // 首字母大写 + 连字符后字母大写
    return parentDir.charAt(0).toUpperCase() + parentDir.slice(1).replace(/-(\w)/g, (_, c) => c.toUpperCase());
  }

  // 转换为大驼峰：首字母大写 + 连字符后字母大写
  return fileName.charAt(0).toUpperCase() + fileName.slice(1).replace(/-(\w)/g, (_, c) => c.toUpperCase());
};

/**
 * 组件自动化注册到全局
 * @param app
 */
export const setupComponents = (app: App) => {
  // 立即导入并打包到主 chunk 中，不进行代码分割，避免异步加载组件导致的性能问题
  const components = import.meta.glob('../components/**/*.vue', { eager: true });
  Object.entries(components).forEach(([path, componentConfig]) => {
    // 获取组件配置
    const component = (componentConfig as { default: ComponentOptions }).default;
    // 全局注册组件
    if (component) {
      const componentName = component.name || convertToCamelCase(path);
      app.component(
        componentName, // 优先使用组件name属性，否则使用路径转换的大驼峰名称
        component
      );
    }
  });
}
```
