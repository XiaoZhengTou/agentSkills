---
name: responsive-layout
description: 为指定页面或组件生成响应式布局方案，覆盖 sm/md/lg/xl 四个断点，输出完整实现代码
agent: frontend
inputs:
  - .claude/project-context.json
outputs:
  - .claude/handoffs/responsive-layout.md
---

# responsive-layout

## description
分析指定页面的布局需求，生成覆盖移动端到桌面端的响应式实现。根据 project-context 自动适配 Tailwind / CSS Grid / Flexbox 方案。

## steps
1. 读取 `.claude/project-context.json`，若不存在则先执行 `/detect-project`
2. 用户指定目标页面或组件，读取现有实现
3. 分析当前布局问题：溢出、固定宽度、缺少断点处理
4. 按断点规划布局变化：
   - sm (640px)：单列，堆叠布局
   - md (768px)：双列，侧边栏折叠
   - lg (1024px)：三列或侧边栏展开
   - xl (1280px)：最大宽度容器，内容居中
5. 输出重构后的布局代码，保留原有业务逻辑不变

## output
```tsx
// 响应式仪表盘布局示例
export function DashboardLayout({ children }: { children: React.ReactNode }) {
  return (
    <div className="min-h-screen bg-gray-50">
      {/* 移动端顶部导航，桌面端隐藏 */}
      <header className="lg:hidden fixed top-0 inset-x-0 h-14 bg-white border-b z-10 flex items-center px-4">
        <MobileMenuButton />
        <Logo className="ml-3" />
      </header>

      <div className="lg:flex">
        {/* 侧边栏：移动端抽屉，桌面端固定 */}
        <aside className="hidden lg:flex lg:flex-col lg:w-64 lg:fixed lg:inset-y-0 border-r bg-white">
          <Sidebar />
        </aside>

        {/* 主内容区 */}
        <main className="lg:pl-64 pt-14 lg:pt-0 w-full">
          <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-6">
            {/* 统计卡片：移动端单列，sm双列，lg四列 */}
            <div className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4 mb-6">
              <StatCard />
              <StatCard />
              <StatCard />
              <StatCard />
            </div>

            {/* 图表区：移动端堆叠，lg并排 */}
            <div className="grid grid-cols-1 lg:grid-cols-3 gap-4">
              <div className="lg:col-span-2">
                <ChartCard />
              </div>
              <div>
                <SummaryCard />
              </div>
            </div>
          </div>
        </main>
      </div>
    </div>
  )
}
```
