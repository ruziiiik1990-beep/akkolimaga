<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>AK-47 Rat Rod Particle Cloud</title>
    <style>
        body { margin: 0; overflow: hidden; background-color: #050505; }
        canvas { display: block; width: 100vw; height: 100vh; }
    </style>
    <!-- Подключаем сам Three.js -->
    <script src="https://cloudflare.com"></script>
</head>
<body>

<canvas class="webgl"></canvas>

<script>
    // 1. Настройки сцены
    const canvas = document.querySelector('canvas.webgl');
    const scene = new THREE.Scene();

    const parameters = {
        size: 0.04, // Размер твоих точек из PointsMaterial
    };

    let geometry = null;
    let material = null;
    let points = null;

    // 2. Функция генерации автомата из пикселей твоей картинки
    const generateWeaponParticles = () => {
        
        // ТВОЙ КОД ОЧИСТКИ ПРЕЖНЕЙ СЦЕНЫ
        if(points !== null) {
            geometry.dispose();
            material.dispose();
            scene.remove(points);
        }

        // Загружаем картинку АК-47 Колымага
        const image = new Image();
        image.crossOrigin = "Anonymous"; // Разрешаем чтение пикселей
        image.src = "https://moy.su";

        image.onload = () => {
            // Создаем невидимый Canvas, чтобы считать цвета пикселей
            const imgCanvas = document.createElement('canvas');
            const ctx = imgCanvas.getContext('2d');
            
            // Уменьшаем масштаб для оптимизации (например, до 150px по ширине)
            const width = 150;
            const height = Math.round((image.height / image.width) * width);
            imgCanvas.width = width;
            imgCanvas.height = height;
            
            ctx.drawImage(image, 0, 0, width, height);
            const imgData = ctx.getImageData(0, 0, width, height).data;

            // Массивы для позиций и цветов частиц
            const positions = [];
            const colors = [];

            // Пробегаемся по пикселям
            for (let y = 0; y < height; y++) {
                for (let x = 0; x < width; x++) {
                    const index = (y * width + x) * 4;
                    const r = imgData[index] / 255;
                    const g = imgData[index + 1] / 255;
                    const b = imgData[index + 2] / 255;
                    const alpha = imgData[index + 3] / 255;

                    // Пропускаем прозрачные пиксели фона
                    if (alpha > 0.1) {
                        // Центрируем и переводим координаты в 3D-пространство Three.js
                        const posX = (x - width / 2) * 0.05;
                        const posY = -(y - height / 2) * 0.05;
                        const posZ = (Math.random() - 0.5) * 0.1; // Небольшая глубина для 3D эффекта

                        positions.push(posX, posY, posZ);
                        colors.push(r, g, b);
                    }
                }
            }

            // Создаем геометрию на основе считанных пикселей
            geometry = new THREE.BufferGeometry();
            geometry.setAttribute('position', new THREE.Float32BufferAttribute(positions, 3));
            geometry.setAttribute('color', new THREE.Float32BufferAttribute(colors, 3));

            // ТВОЙ МАТЕРИАЛ POINTSMATERIAL
            material = new THREE.PointsMaterial({
                size: parameters.size,
                sizeAttenuation: true,
                depthWrite: false,
                blending: THREE.AdditiveBlending,
                vertexColors: true // Включаем цвета пикселей автомата
            });

            // Создаем финальный объект точек и добавляем на сцену
            points = new THREE.Points(geometry, material);
            scene.add(points);
        };
    };

    // Запускаем генерацию
    generateWeaponParticles();

    // 3. Камера и Рендерер
    const sizes = { width: window.innerWidth, height: window.innerHeight };
    const camera = new THREE.PerspectiveCamera(75, sizes.width / sizes.height, 0.1, 100);
    camera.position.z = 5;
    scene.add(camera);

    const renderer = new THREE.WebGLRenderer({ canvas: canvas });
    renderer.setSize(sizes.width, sizes.height);
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));

    // Обновление при изменении размеров экрана
    window.addEventListener('resize', () => {
        sizes.width = window.innerWidth;
        sizes.height = window.innerHeight;
        camera.aspect = sizes.width / sizes.height;
        camera.updateProjectionMatrix();
        renderer.setSize(sizes.width, sizes.height);
    });

    // 4. Анимация бесконечного вращения на 360 градусов
    const clock = new THREE.Clock();

    const tick = () => {
        const elapsedTime = clock.getElapsedTime();

        // Бесконечно крутим облако точек автомата вокруг оси Y
        if(points) {
            points.rotation.y = elapsedTime * 0.5; // Скорость вращения
        }

        renderer.render(scene, camera);
        window.requestAnimationFrame(tick);
    };

    tick();
</script>

</body>
</html>
