from PIL import Image
import os
import tkinter as tk
from tkinter import filedialog, simpledialog
import io
import re

def extract_number_from_filename(filename):
    """Извлекает число из имени файла. Предполагается, что имя файла имеет формат 'xx.ext'."""
    match = re.search(r'(\d+)', filename)
    return int(match.group()) if match else float('inf')

def merge_images(image_paths):
    # Сортируем пути к файлам по числовому значению в имени
    image_paths = list(image_paths)  # Преобразуем кортеж в список, если это кортеж
    image_paths.sort(key=lambda x: extract_number_from_filename(os.path.basename(x)))

    if not image_paths:
        print("No images provided for merging.")
        return None

    # Открываем все изображения и получаем их размеры
    try:
        opened_images = [Image.open(image) for image in image_paths]
    except Exception as e:
        print(f"Error opening images: {e}")
        return None

    widths, heights = zip(*(img.size for img in opened_images))

    total_height = sum(heights)
    max_width = max(widths)

    # Создаем новое изображение с необходимыми размерами
    merged_image = Image.new('RGB', (max_width, total_height))

    y_offset = 0
    for img in opened_images:
        merged_image.paste(img, (0, y_offset))
        y_offset += img.height

    # Сохраняем изображение в байтовый поток
    img_byte_arr = io.BytesIO()
    merged_image.save(img_byte_arr, format='PNG')
    img_byte_arr.seek(0)  # Возвращаем указатель в начало потока
    return img_byte_arr

def main():
    root = tk.Tk()
    root.withdraw()  # Скрыть основное окно

    # Выбор нескольких изображений
    file_paths = filedialog.askopenfilenames(
        title="Выберите изображения для объединения",
        filetypes=[("Image Files", "*.png;*.jpg;*.jpeg;*.webp")]
    )
    if not file_paths:
        return

    # Ввод названия итогового файла
    output_file = simpledialog.askstring("Название файла", "Введите название файла (без расширения):")
    if not output_file:
        return

    # Выбор папки, в которую можно сохранить итоговое изображение
    output_save_folder = filedialog.askdirectory(title="Выберите папку для сохранения файла")
    if not output_save_folder:
        return

    # Объединение изображений и сохранение в байтовый поток
    img_byte_arr = merge_images(file_paths)
    if img_byte_arr is None:
        return

    # Путь для сохранения объединенного изображения
    output_path = os.path.join(output_save_folder, f"{output_file}.png")

    # Сохранение объединенного изображения непосредственно на диск
    try:
        with open(output_path, 'wb') as f:
            f.write(img_byte_arr.getvalue())
        print(f'Image saved to {output_path}')
    except Exception as e:
        print(f'Error saving image: {e}')

if __name__ == "__main__":
    main()
