# fbx-viewer-vue

Simple and fast strengthen your apps with 3d

## Component Parameters

| Name               | Type                                          | Default Value                                    | Description                                   |
|--------------------|-----------------------------------------------|--------------------------------------------------|-----------------------------------------------|
| `fbx`              | String                                        | -                                                | Path to the FBX file to be displayed.         |
| `ToplightPosition` | { x: number; y: number; z: number }           | `{ x: 0, y: 2, z: 0 }`                           | Position of the top light in 3D space.       |
| `TopPointLight`    | { color: string; intensity: number; distance: number } | `{ color: '#ffffff', intensity: 1, distance: 8 }` | Properties of the top point light.           |
| `BottomLightPosition` | { x: number; y: number; z: number }         | `{ x: 0, y: -2, z: 0 }`                          | Position of the bottom light in 3D space.    |
| `BottomPointLight` | { color: string; intensity: number; distance: number } | `{ color: '#421630', intensity: 1, distance: 10 }` | Properties of the bottom point light.        |
| `LeftLightPosition` | { x: number; y: number; z: number }         | `{ x: 0, y: -2, z: 0 }`                          | Position of the left light in 3D space.     |
| `LeftPointLight`   | { color: string; intensity: number; }        | `{ color: '#ffffff', intensity: 1, distance: 10 }` | Properties of the left point light.          |
| `RightLightPosition` | { x: number; y: number; z: number }        | `{ x: 0, y: -2, z: 0 }`                          | Position of the right light in 3D space.    |
| `RightPointLight`  | { color: string; intensity: number; }        | `{ color: '#ffffff', intensity: 1, distance: 10 }` | Properties of the right point light.         |
| `ObjColor`         | String                                        | `'#fefefe'`                                      | Default color of the object in the scene.    |
| `SceneBg`          | { color: string; alpha: number }              | `{ color: '#041839', alpha: 0 }`                 | Background color of the scene.               |
| `animate`          | { animate: boolean; duration: number }        | `{ animate: true, duration: 1000 }`              | Animation settings for the viewer.           |
| `distance`         | { minDistance: number; maxDistance: number }  | `{ minDistance: 0.8, maxDistance: 1 }`           | Distance constraints for camera movement.    |
| `autoRotate`       | { value: boolean; rotateSpeed: number }       | `{ value: true, rotateSpeed: 2 }`                | Auto-rotation settings for the scene.        |
| `cameraPosition`   | { x: number; y: number; z: number }           | `{ x: 1, y: 1, z: 1 }`                           | Default position of the camera.             |
