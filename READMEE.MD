# Expo Tutorial
## Введение

Речь идёт о разработке приложения **StickerSmash** — оно работает на Android, iOS и в браузере из одного и того же кода. Задача туториала — освоить основной набор инструментов Expo SDK:

- создание проекта на базовом шаблоне с TypeScript;
- навигация через Expo Router (стек экранов + нижние вкладки);
- построение интерфейса на Flexbox;
- выбор фото из системной галереи;
- модальное окно для выбора стикера (`<Modal>` + `<FlatList>`);
- жесты (тап, перетаскивание) для взаимодействия со стикером;
- сохранение результата в виде файла-изображения;
- учёт различий между платформами;
- финальная настройка статус-бара, иконки и заставки.

Стартовый пример, с которого обычно начинают знакомство с React Native:

```tsx
import { StyleSheet, Text, View } from 'react-native';

export default function Index() {
  return (
    <View style={styles.wrapper}>
      <Text>Hello world!</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  wrapper: { flex: 1, alignItems: 'center', justifyContent: 'center' },
});
```

---

## Создайте свое первое приложение

Проект создаётся командой `create-expo-app` — она генерирует папку с готовой структурой, где уже настроены Expo Router, TypeScript и поддержка трёх платформ одновременно. Стили в React Native — это не CSS-файлы, а обычные JS-объекты, которые описываются через `StyleSheet.create()`.

### Шаги

1. Создание и запуск проекта:

```bash
npx create-expo-app@latest StickerSmash
cd StickerSmash
npx expo start
```

2. После скачивания архива с картинками — заменить стандартные файлы в `assets/images` своими.

3. Очистка шаблона от демонстрационного кода:

```bash
npm run reset-project
```

После этой команды в `src/app` остаются только `index.tsx` и `_layout.tsx`, а старые файлы переносятся в папку `example`.

4. Правки в стартовом экране:

```tsx
import { Text, View, StyleSheet } from 'react-native';

export default function Index() {
  return (
    <View style={styles.wrapper}>
      <Text style={styles.title}>Главный экран</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  wrapper: { flex: 1, backgroundColor: '#1e2126', alignItems: 'center', justifyContent: 'center' },
  title: { color: '#fff' },
});
```

---

## Добавить навигацию
Expo Router строит маршруты на основе файловой структуры: каждый файл в папке app автоматически становится отдельным экраном. Базовые правила:
* _layout.tsx — общий каркас для дочерних экранов (шапка, табы).
* index.tsx — стартовый экран, соответствует адресу /.
* +not-found.tsx — заглушка для несуществующего маршрута.
* (tabs) — группировка экранов без влияния на URL.

### Стек-навигатор

Компонент `<Stack>` отвечает за переходы вперёд/назад между экранами:

```tsx
import { Stack } from 'expo-router';

export default function RootLayout() {
  return (
    <Stack>
      <Stack.Screen name="index" options={{ title: 'Главная' }} />
      <Stack.Screen name="about" options={{ title: 'О приложении' }} />
    </Stack>
  );
}
```

### Переход между экранами

```tsx
import { Link } from 'expo-router';

<Link href="/about">Перейти на страницу "О приложении"</Link>
```

### Маршрут для 404

Файл `+not-found.tsx` показывает запасной экран, если пользователь попал на несуществующий адрес — вместо аварийного завершения приложения.

### Нижняя панель вкладок

Экраны переносятся в подпапку `(tabs)`, для неё создаётся отдельный layout:

```tsx
import { Tabs } from 'expo-router';
import Ionicons from '@expo/vector-icons/Ionicons';

export default function TabsLayout() {
  return (
    <Tabs screenOptions={{ tabBarActiveTintColor: '#f2c94c' }}>
      <Tabs.Screen
        name="index"
        options={{
          title: 'Главная',
          tabBarIcon: ({ color }) => <Ionicons name="home" size={22} color={color} />,
        }}
      />
    </Tabs>
  );
}
```

Библиотека `@expo/vector-icons` устанавливается отдельно и содержит сразу несколько популярных наборов значков.

---

## Постройте экран

Экран удобно раскладывать на простые блоки: сверху — изображение, снизу — пара кнопок. Для изображений применяется кроссплатформенный компонент `<Image>` из `expo-image`, который принимает либо локальный файл (`require`), либо адрес в сети (`uri`).

Для кликабельных элементов вместо стандартного `<Button>` обычно берут **`<Pressable>`** — он различает не только простое нажатие, но и долгое удержание, момент "нажато"/"отпущено".

### Показ изображения

```tsx
import { Image } from 'expo-image';

const bgImage = require('@/assets/images/background-image.png');

<Image source={bgImage} style={{ width: 300, height: 400, borderRadius: 16 }} />
```

### Компонент кнопки

```tsx
import { Pressable, Text, View } from 'react-native';

type Props = { label: string; onPress?: () => void };

export default function AppButton({ label, onPress }: Props) {
  return (
    <View>
      <Pressable onPress={onPress} style={{ padding: 12 }}>
        <Text style={{ color: '#fff' }}>{label}</Text>
      </Pressable>
    </View>
  );
}
```

Такие "неэкранные" компоненты выносят в отдельную папку `components`, а не держат внутри `app` — иначе Expo Router попытается воспринять их как отдельные маршруты.

### Второй вариант кнопки (primary)

Чтобы у кнопок был разный внешний вид, добавляют проп-переключатель темы:

```tsx
type Props = { label: string; theme?: 'primary' };

export default function AppButton({ label, theme }: Props) {
  const isPrimary = theme === 'primary';
  return (
    <Pressable style={{ backgroundColor: isPrimary ? '#fff' : 'transparent', borderRadius: 10 }}>
      <Text style={{ color: isPrimary ? '#000' : '#fff' }}>{label}</Text>
    </Pressable>
  );
}
```

---

## Используйте набор изображений

Базовые компоненты React Native не умеют открывать системную галерею — для этого подключается библиотека **`expo-image-picker`**.

```bash
npx expo install expo-image-picker
```

### Выбор фото

Метод `launchImageLibraryAsync` открывает системный интерфейс выбора и возвращает объект с массивом `assets`:

```tsx
import * as ImagePicker from 'expo-image-picker';
import { useState } from 'react';

export default function Index() {
  const [photoUri, setPhotoUri] = useState<string | undefined>();

  const choosePhoto = async () => {
    const res = await ImagePicker.launchImageLibraryAsync({
      mediaTypes: ['images'],
      quality: 1,
    });
    if (!res.canceled) {
      setPhotoUri(res.assets[0].uri);
    } else {
      alert('Изображение не выбрано.');
    }
  };

  // ...остальной JSX
}
```

Если результат `canceled: false`, из `res.assets[0].uri` берётся путь к файлу и сохраняется в состоянии компонента (`useState`). Дальше это значение передаётся в компонент показа изображения — вместо картинки-заглушки отображается уже выбранное фото.

---

## Создайте модаль

`<Modal>` — стандартный компонент React Native для показа контента поверх остального интерфейса. Основные пропсы:

- `visible` — открыта модалка или нет;
- `transparent` — прозрачный фон или на весь экран;
- `animationType` — способ появления (`slide`, `fade`, `none`).

### Компонент модалки

```tsx
import { Modal, View, Text, Pressable } from 'react-native';

type Props = { open: boolean; onClose: () => void; children: React.ReactNode };

export default function StickerPicker({ open, onClose, children }: Props) {
  return (
    <Modal visible={open} transparent animationType="slide">
      <View style={{ backgroundColor: '#25292e', padding: 16 }}>
        <Pressable onPress={onClose}>
          <Text style={{ color: '#fff' }}>Закрыть</Text>
        </Pressable>
        {children}
      </View>
    </Modal>
  );
}
```

### Список стикеров

Для перебора вариантов используется `<FlatList>` — он рендерит только видимые элементы, что экономит ресурсы на больших списках:

```tsx
import { FlatList, Pressable, Image } from 'react-native';

<FlatList
  horizontal
  data={stickers}
  keyExtractor={(_, i) => String(i)}
  renderItem={({ item }) => (
    <Pressable onPress={() => selectSticker(item)}>
      <Image source={item} style={{ width: 90, height: 90 }} />
    </Pressable>
  )}
/>
```

### Показ выбранного стикера

Выбор сохраняется в состоянии (`useState`), а затем рендерится поверх основного изображения отдельным компонентом-наклейкой — условно, только если стикер уже выбран (`pickedSticker && <Sticker .../>`).

---

## Добавить жесты

Для распознавания касаний применяется **React Native Gesture Handler**, а для плавных изменений значений — **Reanimated**. Всё дерево компонентов должно быть обёрнуто в `<GestureHandlerRootView>` — без этого жесты не будут работать.

Центральное понятие Reanimated — «общие значения» (`shared values`), которые можно менять напрямую, минуя обычный цикл рендеринга React.

### Жест двойного тапа (масштаб)

```tsx
import { useSharedValue, useAnimatedStyle, withSpring } from 'react-native-reanimated';
import { Gesture, GestureDetector } from 'react-native-gesture-handler';
import Animated from 'react-native-reanimated';

function ResizableSticker({ size }: { size: number }) {
  const scale = useSharedValue(size);

  const doubleTap = Gesture.Tap()
    .numberOfTaps(2)
    .onStart(() => {
      scale.value = scale.value === size ? size * 2 : size;
    });

  const style = useAnimatedStyle(() => ({
    width: withSpring(scale.value),
    height: withSpring(scale.value),
  }));

  return (
    <GestureDetector gesture={doubleTap}>
      <Animated.Image source={/* ... */} style={style} />
    </GestureDetector>
  );
}
```

### Жест перетаскивания (pan)

```tsx
const x = useSharedValue(0);
const y = useSharedValue(0);

const pan = Gesture.Pan().onChange((e) => {
  x.value += e.changeX;
  y.value += e.changeY;
});

const wrapperStyle = useAnimatedStyle(() => ({
  transform: [{ translateX: x.value }, { translateY: y.value }],
}));
```

Оба жеста оборачивают нужный элемент компонентом `<GestureDetector gesture={...}>`; при желании несколько жестов комбинируют через `Gesture.Simultaneous()`.

---

## Сделайте скриншот

Чтобы сохранить видимую часть экрана как картинку, используются две библиотеки:

- **`react-native-view-shot`** — делает снимок указанного `<View>`;
- **`expo-media-library`** — сохраняет готовый файл в галерею устройства.

```bash
npx expo install react-native-view-shot expo-media-library
```

### Запрос разрешения

Доступ к медиатеке — чувствительное разрешение, поэтому перед сохранением его нужно запросить:

```tsx
const [permission, requestPermission] = ImagePicker.useMediaLibraryPermissions();

useEffect(() => {
  if (!permission?.granted) requestPermission();
}, []);
```

### Ссылка на область захвата

Область, которую нужно сфотографировать, помечается через `ref` с обязательным параметром `collapsable={false}` — иначе React Native может "схлопнуть" этот `View` при оптимизации, и снимок не получится:

```tsx
const viewRef = useRef<View>(null);
```

### Сохранение

```tsx
const saveResult = async () => {
  try {
    const uri = await captureRef(viewRef, { height: 440, quality: 1 });
    await MediaLibrary.saveToLibraryAsync(uri);
    alert('Сохранено!');
  } catch (e) {
    console.log(e);
  }
};
```

---

## Различия по платформам управления

Не все библиотеки одинаково работают на всех платформах: `react-native-view-shot` рассчитан только на Android и iOS, для веба нужен другой инструмент — **`dom-to-image`**, который превращает DOM-элемент в изображение прямо в браузере.

Определить текущую платформу помогает модуль `Platform` из `react-native` — свойство `Platform.OS` возвращает `'ios'`, `'android'` или `'web'`.

### Ветвление логики по платформе

```tsx
import { Platform } from 'react-native';
import domtoimage from 'dom-to-image';

const saveResult = async () => {
  if (Platform.OS === 'web') {
    const dataUrl = await domtoimage.toJpeg(viewRef.current);
    const link = document.createElement('a');
    link.href = dataUrl;
    link.download = 'result.jpeg';
    link.click();
  } else {
    const uri = await captureRef(viewRef);
    await MediaLibrary.saveToLibraryAsync(uri);
  }
};
```

### Типы для dom-to-image

У библиотеки нет готовых типов для TypeScript, поэтому модуль объявляется вручную в файле `types.d.ts`:

```ts
declare module 'dom-to-image';
```

---

## Настройте строку статуса, заставку и иконку
### Теория

Перед публикацией приложения обычно настраивают три визуальных элемента.

### Статус-бар

Библиотека `expo-status-bar` уже включена в стандартный шаблон, остаётся только задать стиль текста (светлый или тёмный):

```tsx
import { StatusBar } from 'expo-status-bar';

<StatusBar style="light" />
```

### Иконка приложения

Задаётся квадратным файлом `assets/images/icon.png` (1024×1024 px), путь к которому прописан в `app.json` в поле `"icon"`. По умолчанию менять ничего не требуется — достаточно заменить сам файл.

### Заставка (splash screen)

Настраивается через плагин `expo-splash-screen` в `app.json`:

```json
{
  "plugins": [
    ["expo-splash-screen", { "image": "./assets/images/splash-icon.png" }]
  ]
}
```

Важный момент: заставку нельзя увидеть в Expo Go — она отображается только в preview- или production-сборке приложения.

---

## Учебные материалы

После завершения туториала для закрепления материала рекомендуется:

- **React** — основы и хуки (`useState`, `useEffect` и другие) из официальной документации React.
- **React Native** — базовые компоненты (`View`, `Text`), платформенный код, Flexbox-вёрстка, работа со списками.
- **Expo Router** — более глубокое изучение файловой маршрутизации: вложенные группы, модальные маршруты, параметры в адресе.
- **Жесты и анимации** — документация React Native Gesture Handler и React Native Reanimated.
- **Сборка и публикация приложения** — как подготовить и выложить приложение в App Store и Google Play.
- **Отладка** — инструменты для поиска и исправления ошибок во время работы приложения.
