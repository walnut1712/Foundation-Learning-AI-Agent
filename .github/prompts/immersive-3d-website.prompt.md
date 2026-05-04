---
description: "Build an interactive 3D website with Three.js or WebGL featuring responsive design, camera controls, lighting, textures, animations, and accessibility"
name: "Immersive 3D Website"
argument-hint: "Describe your 3D experience goal (e.g., product showcase, interactive model, environment)"
agent: "agent"
---

# Immersive 3D Website Development

Create an interactive, performant 3D website experience with professional structure and user engagement.

## 1. Project Structure & Setup

- **Initialize project** with clean folder organization
- **Choose framework**: Three.js (easier, well-documented) or raw WebGL (advanced)
- **Configure build tools**: Webpack/Vite for bundling, asset management
- **Version control**: .gitignore for node_modules, build outputs
- **Document dependencies** in package.json with clear descriptions

## 2. 3D Scene Foundation

- **Create main scene** with camera, renderer, and lighting
- **Set up viewport** responsive to window size
- **Initialize Three.js** with WebGL renderer or choose WebGL context
- **Configure canvas** for proper aspect ratio and pixel density
- **Test rendering** with basic geometry before adding complexity

## 3. 3D Models & Assets

- **Load or create geometry**: Use models (gltf/glb files) or procedurally generate shapes
- **Apply materials**: Standard, physical, or custom shaders as needed
- **Add textures**: Diffuse, normal, roughness maps with proper wrapping
- **Optimize file sizes**: Compress models, use appropriate texture resolution
- **Handle asset loading**: Progress tracking, error handling

## 4. Lighting & Visual Design

- **Implement light sources**: Ambient, directional, point, or spot lights
- **Configure shadows**: Shadow maps, light depth, quality settings
- **Set tone**: Color palette, exposure, post-processing effects
- **Create depth**: Use layering, fog, or depth of field
- **Preview iterations**: Render passes, wireframe debugging

## 5. Camera & Interactions

- **Set up camera controls**:
  - Orbit controls (rotate around object)
  - First-person controls (WASD movement)
  - Manual camera positioning
- **Handle user input**: Mouse, keyboard, touch events
- **Add responsiveness**: Resize event handling, mobile touch gestures
- **Smooth animations**: Camera tweening, easing functions

## 6. Animations & Motion

- **Define animation loops**: RequestAnimationFrame for smooth 60fps
- **Animate objects**: Position, rotation, scale over time
- **Use tweening libraries**: Smooth transitions with easing
- **Sync animations**: State management for coordinated movement
- **Performance**: Throttle updates, use delta time for frame-rate independence

## 7. User Interactions

- **Clickable objects**: Raycasting for intersection detection
- **UI controls**: Buttons, sliders for parameter adjustment
- **Feedback systems**: Hover states, selection highlighting, tooltips
- **Event handling**: Mouse, click, drag, scroll events
- **Mobile support**: Touch gestures, viewport optimization

## 8. Performance Optimization

- **Geometry optimization**: Reduce polygon count, use LOD (Level of Detail)
- **Texture optimization**: Compressed formats, appropriate resolution
- **Renderer settings**: Pixel ratio, anti-aliasing, render target scaling
- **Caching**: Reuse materials, geometries, textures
- **Profiling**: Monitor FPS, GPU usage, memory consumption

## 9. Accessibility Considerations

- **Keyboard navigation**: Tab through interactive elements, arrow keys for controls
- **Screen reader support**: ARIA labels, semantic HTML structure
- **Color contrast**: Sufficient contrast in UI overlays
- **Motion sensitivity**: Respect prefers-reduced-motion for animations
- **Text alternatives**: Descriptions for 3D content
- **Mobile accessibility**: Touch target sizes, sensible defaults

## 10. Code Quality & Documentation

- **Modular architecture**: Separate concerns (scene, camera, controls, renderer)
- **Clean code**: Consistent naming, comments on complex logic
- **Error handling**: Graceful fallbacks, console warnings
- **Testing**: Unit tests for utilities, visual regression tests
- **README**: Setup instructions, controls guide, browser requirements

## 11. Deployment & Distribution

- **Build process**: Minification, tree-shaking for production
- **Asset hosting**: CDN for models, textures, or embedded in bundle
- **Browser compatibility**: Test on major browsers, mobile devices
- **Performance budget**: Target frame rate (60fps), load time (<3s)
- **Monitoring**: Error tracking, analytics for user engagement

## Output Requirements

- **Deliverables**: Source code, production build, README
- **Documentation**: Setup instructions, controls/interactions guide
- **Performance metrics**: FPS target achieved, bundle size
- **Browser support**: List of tested environments
- **Code structure**: Clear organization suitable for portfolio/coursework

**Style**: Professional, performant, accessible, production-ready
