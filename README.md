import os
import shutil
import requests
import pyautogui as pag
from bs4 import BeautifulSoup
import pyperclip  # Для буфера обмена
from IP import API, KEY  # Импорт ключей API и KEY
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from webdriver_manager.chrome import ChromeDriverManager
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support.ui import Select
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.common.action_chains import ActionChains
from selenium.webdriver.common.keys import Keys
# You can install Pillow (the PIL fork) using:
# pip install Pillow

from PIL import Image  # Импортируем Image из PIL

# Инициализация WebDriver
driver = webdriver.Chrome(service=Service(ChromeDriverManager().install()))


# Функция для загрузки файла по ссылке
def download_file(url, filename):
    try:
        response = requests.get(url, stream=True)
        response.raise_for_status()
        with open(filename, 'wb') as f:
            for chunk in response.iter_content(chunk_size=8192):
                f.write(chunk)
        print(f'Файл {filename} успешно сохранён.')
    except requests.exceptions.RequestException as e:
        print(f'Произошла ошибка при загрузке файла: {e}')

# Функция для получения URL изображений
def get_image_urls(url):
    try:
        response = requests.get(url)
        response.raise_for_status()
        soup = BeautifulSoup(response.text, 'html.parser')
        images = soup.select('button[data-fancybox="gallery"] img')[:11]
        return [img['src'] for img in images if 'src' in img.attrs]  # Проверка наличия атрибута
    except requests.exceptions.RequestException as e:
        print(f'Ошибка при получении страницы: {e}')
        return []
    except Exception as e:
        print(f'Ошибка: {e}')
        return []

# Функция для получения информации о продукте
def get_product_info(url):
    # Установите заголовки
    headers = {
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36'
    }

    # Выполните GET-запрос
    response = requests.get(url, headers=headers)

    # Проверьте ответ
    if response.ok:
        print("Запрос успешен")
        soup = BeautifulSoup(response.text, 'html.parser')

        # Извлеките название продукта
        title = soup.find('h1')
        title_text = title.text.strip() if title else "Название отсутствует."  # Обработка NoneType

        # Извлеките цену
        price = soup.find('span', class_='styles_sidebar__main__DaXQC')
        price_text = price.text.strip() if price else 'Цена отсутствует.'  # Обработка NoneType

        # Извлеките описание
        description_div = soup.find('div', class_='styles_description__8_RRa')
        description_text = description_div.text.strip() if description_div else 'Описание отсутствует.'  # Обработка NoneType
        print("Информация скопирована в буфер обмена:\n", title_text, price_text, description_text)
        return title_text, price_text, description_text
    else:
        print(f"Произошла ошибка: {response.status_code}")
        return None, None, None

# Инициализация WebDriver
driver = webdriver.Chrome()  # Убедитесь, что у вас правильно установлен ChromeDriver
try:
    # Переход к странице профиля
    profile_url = "https://999.md/ru/profile/EgorCeban"
    driver.get(profile_url)

    conf = pag.confirm("Продолжить?", "Confirmation")
    if conf == "OK":
        # Получение обновленного URL
        updated_url = driver.current_url  # Получаем текущий URL из браузера

        print(f"Используемый URL: {updated_url}")

        # Получаем данные о продукте
        title_text, price_text, description_text = get_product_info(updated_url)

        if title_text is None or price_text is None or description_text is None:
            print("Не удалось получить информацию о продукте.")

        # Получение изображений
        image_urls = get_image_urls(updated_url)

        if image_urls:
            print(f'Найдено {len(image_urls)} изображений.')
            directory = 'downloaded_images'

            if os.path.exists(directory):
                shutil.rmtree(directory)
            os.makedirs(directory)
            for i, img_url in enumerate(image_urls):
                # Преобразуем относительный путь к абсолютному URL, если нужно
                if not img_url.startswith('http'):
                    base_url = updated_url.rsplit('/', 1)[0] + "/"
                    img_url = base_url + img_url

                filename = os.path.join(directory, f'image_{i + 1}.jpg')
                download_file(img_url, filename)
        else:
            print('Изображения не найдены.')
        clipboard_text = pyperclip.paste()  # Получаем текст из буфера обмена

    # Получаем текст из буфера обмена для вставки
except Exception as e:
    print(f"An error occurred: {str(e)}")
    print("Current URL:", driver.current_url)

try:
    # Получаем текст из буфера обмена для вставки
    clipboard_text = pyperclip.paste()  # Получаем текст из буфера обмена
    # Open the login page of the website
    driver.get("https://v2.simpalsid.com/sid/ru/user/login")  # Replace with actual login URL if needed

    # Wait for the login input field and enter the username
    username_input = WebDriverWait(driver, 10).until(
        EC.presence_of_element_located((By.CSS_SELECTOR, "input[placeholder='Введите логин или e-mail']"))
    )
    username_input.send_keys(API)

    # Wait for the password input field and enter the password
    password_input = WebDriverWait(driver, 10).until(
        EC.presence_of_element_located((By.CSS_SELECTOR, "input[placeholder='Введите от 6 символов']"))
    )
    password_input.send_keys(KEY)
    pag.sleep(5)  # Ждем, чтобы устранить возможные задержки

    # Wait for the "Войти" button and click it
    login_button = WebDriverWait(driver, 10).until(
        EC.element_to_be_clickable((By.CSS_SELECTOR, "button.Button_solid__gEcaH"))
    )
    login_button.click()

    # Проверяем и нажимаем кнопку входа
    driver.execute_script("arguments[0].click();", login_button)

    # Опционально, ждём появления элемента после входа (например, главной страницы)

    # Ждем, пока не появится нужная ссылка на странице
    link = WebDriverWait(driver, 10).until(
        EC.element_to_be_clickable((By.CSS_SELECTOR, "a.styles_redirectBtn__o_7FD"))
    )
    link.click()

    # Wait for the redirect to the 999.md URL
    WebDriverWait(driver, 20).until(EC.url_contains("999.md"))

    # Navigate to the profile page
except Exception as e:
    print(f"An error occurred: {str(e)}")
    print("Current URL:", driver.current_url)

path = ["wash.png", "freeze.png", "/Users/egorceban/PycharmProjects/pythonProject/brave3690/oven.png", "micro.png", "dish.png", "coffee.png"] # Картинки
Wash = r"/Users/egorceban/PycharmProjects/pythonProject/wash.png"
def find_and_click(image_path):
    try:
        location = pag.locateOnScreen(image_path, confidence=0.11)
        pag.sleep(1)
        if location is not None:
            print(f"Кнопка найдена!")
        else:
            print(f"Кнопка '{image_path}' не найдена.")
    except Exception as e:
        print(f"Ошибка: {e}")
        return False  # Возвращаем False, если произошла ошибка

# Основная функция
def main():
    while True:
        try:
                location = pag.locateOnScreen(Wash, confidence=0.12)
                if location is not None:
                    print(f"Кнопка Wash найдена")
                    # Переход к странице добавления объявления
                    url = "https://999.md/ru/add?category=household-appliances&subcategory=household-appliances%2Fwashing-machines"
                    driver.get(url)
                    pag.sleep(2)

                    try:
                        close_button = WebDriverWait(driver, 5).until(
                            EC.element_to_be_clickable((By.CSS_SELECTOR, "a.introjs-button.introjs-nextbutton.introjs-donebutton"))
                        )
                        close_button.click()
                        driver.refresh()  # Автоматическое обновление браузера
                        pag.sleep(2)  # Задержка после обновления  
                        print("Закрыто окно 'Понятно'.")
                    except Exception:
                        pass
                        try:

                        # Попробуем кликнуть по элементу перед вводом текста с использованием JavaScript для надежного клика
                            title_input = WebDriverWait(driver, 50).until(
                            EC.element_to_be_clickable((By.CSS_SELECTOR, "input[name='#12.value.ru']"))
                        )

                            print("Поле ввода заголовка найдено.")
                            driver.execute_script("arguments[0].click();", title_input)
                            print("Сработал метод: JavaScript click на заголовок")
                            title_input.send_keys(title_text)  # Вставляем текст
                            title_input.send_keys(Keys.TAB)  # Перемещаем фокус к следующему элементу
                            pyperclip.copy(description_text)
                            driver.switch_to.active_element.send_keys(pyperclip.paste())  # Вставляем текст описания
                        except Exception as e:
                            print("Не удалось найти или кликнуть по полю ввода заголовка:", e)
                            title_input = None

           
                        # Попробуем кликнуть по элементу перед вводом текста            
                        try:
                            dropdown = WebDriverWait(driver, 10).until(
                                EC.element_to_be_clickable((By.CSS_SELECTOR, ".style_select__input__h0wAV"))
                            )
                            dropdown.click()
                            print("Выпадающий список найден и открыт.")
                        except Exception as e:
                            print("Ошибка при открытии выпадающего списка:", e)


                        try:
                            upload_label = WebDriverWait(driver, 10).until(
                                EC.presence_of_element_located((By.CSS_SELECTOR, "label[for='upload-photo']"))
                            )
                            # Первая попытка: JavaScript click
                            try:
#                                driver.execute_script("arguments[0].click();", upload_label)
                                print("Сработал метод: JavaScript click на upload-label")
                            except Exception as js_e:
                                print("Метод click через JavaScript не удался, пробуем ActionChains:", js_e)
                                # Вторая попытка: ActionChains click
                                try:
#                                    ActionChains(driver).move_to_element(upload_label).click().perform()
                                    print("Сработал метод: ActionChains click на upload-label")
                                except Exception as ac_e:
                                    print("Метод ActionChains click не удался:", ac_e)
                        except Exception as e:
                            print("Не удалось найти элемент upload-photo:", e)

                        # Выбор из выпадающего списка "Кишинёв мун." с проверкой
                        try:
                            select_elem = WebDriverWait(driver, 10).until(
                                EC.presence_of_element_located((By.CSS_SELECTOR, "select.style_select__input__h0wAV[name='#7.value']"))
                            )
                            dropdown = Select(select_elem)
                            dropdown.select_by_visible_text("Кишинёв мун.")
                            print("Выбран пункт 'Кишинёв мун.' из выпадающего списка")
                            
                            # Перемещаем фокус и вводим цену, используя активный элемент
                            select_elem.send_keys(Keys.TAB)  
                            active = driver.switch_to.active_element
                            pyperclip.copy(price_text)
                            active.send_keys(pyperclip.paste())
                            
                            # Переход к следующему элементу с помощью ActionChains
                            ActionChains(driver).send_keys(Keys.TAB).perform()

                            # Навигация по опциям с ARROW_DOWN и ENTER через активный элемент
                            active = driver.switch_to.active_element
                            active.send_keys(Keys.ARROW_DOWN)
                            active.send_keys(Keys.ARROW_DOWN)
                            active.send_keys(Keys.ARROW_DOWN)
                            active.send_keys(Keys.ENTER)
                            
                            # Если нужно, повторяем переход фокуса и навигацию
                            active.send_keys(Keys.TAB)
                            active.send_keys(Keys.TAB)
                            active.send_keys(Keys.ARROW_DOWN)
                            active.send_keys(Keys.ARROW_DOWN)
                            active.send_keys(Keys.ENTER)
                            
                            active.send_keys(Keys.TAB)
                            active.send_keys(Keys.ARROW_DOWN)
                            active.send_keys(Keys.ARROW_DOWN)
                            active.send_keys(Keys.ENTER)
                            
                            # Заканчиваем переключение, перемещаясь к следующему элементу
                            active.send_keys(Keys.TAB)
                            active.send_keys(Keys.TAB)
                            active.send_keys(Keys.ARROW_DOWN)
                            active.send_keys(Keys.ARROW_DOWN)
                            active.send_keys(Keys.ARROW_DOWN)
                            active.send_keys(Keys.ARROW_DOWN)
                            active.send_keys(Keys.ENTER)
                        except Exception as e:
                            print("Ошибка при выборе из выпадающего списка:", e)
                        except Exception as e:
                            print("Ошибка при выборе из выпадающего списка:", e)
                        # Попытка 1: Click через JavaScript
                        # Ждем поле для названия и вставляем название
                        # Попытка 2: Click через ActionChains
                        try:
                            upload_label_ac = WebDriverWait(driver, 10).until(
                                EC.element_to_be_clickable((By.CSS_SELECTOR, "label[for='upload-photo']"))
                            )

                            ActionChains(driver).move_to_element(upload_label_ac).click().perform()
                            print("Сработал метод: ActionChains click на upload-label")
                            conf = pag.confirm("Продолжить?", "Confirmation")

                        except Exception as e:
                            print("Метод ActionChains click не удался:", e)
                else:
                    print(f"Кнопка найдена?")
                    return True

        except Exception:

            try:
                location = pag.locateOnScreen(Wash, confidence=0.12)
                if location is not None:
                    print(f"Кнопка найдена")
                    # Переход к странице добавления объявления
                    url = "https://999.md/ru/add?category=household-appliances&subcategory=household-appliances%2Fwashing-machines"
                    driver.get(url)
                    pag.sleep(2)

                    try:
                        close_button = WebDriverWait(driver, 5).until(
                            EC.element_to_be_clickable((By.CSS_SELECTOR, "a.introjs-button.introjs-nextbutton.introjs-donebutton"))
                        )
                        close_button.click()
                        driver.refresh()  # Автоматическое обновление браузера
                        pag.sleep(1)  # Задержка после обновления  
                        print("Закрыто окно 'Понятно'.")
                    except Exception:
                        pass
                        try:
                            title_input = WebDriverWait(driver, 50).until(
                                EC.element_to_be_clickable((By.CSS_SELECTOR, "div.style_input__box__yfrHZ > input.style_input__input__YomV1"))
                            )
                            print("Поле ввода заголовка найдено.")
                            title_input.click()  # Кликаем по элементу
                            title_input.send_keys(title_text)  # Вставляем текст
                            title_input.send_keys(Keys.TAB)  # Перемещаем фокус к следующему элементу   
                            driver.switch_to.active_element.send_keys(description_text)  # Вставляем текст описания
                            print("Текст описания вставлен в поле ввода.")
                        except Exception as e:
                            print("Не удалось найти или кликнуть по полю ввода заголовка:", e)
                            title_input = None  
                        # Попробуем кликнуть по элементу перед вводом текста
                        try:
                            dropdown = WebDriverWait(driver, 10).until(
                                EC.element_to_be_clickable((By.CSS_SELECTOR, ".style_select__input__h0wAV"))
                            )
                            dropdown.click()
                            print("Выпадающий список найден и открыт.")
                        except Exception as e:
                            print("Ошибка при открытии выпадающего списка:", e)

                        try:
                            upload_label = WebDriverWait(driver, 10).until(
                                EC.presence_of_element_located((By.CSS_SELECTOR, "label[for='upload-photo']"))
                            )
                            # Первая попытка: JavaScript click
                            try:
#                                driver.execute_script("arguments[0].click();", upload_label)
                                print("Сработал метод: JavaScript click на upload-label")
                            except Exception as js_e:
                                print("Метод click через JavaScript не удался, пробуем ActionChains:", js_e)
                                # Вторая попытка: ActionChains click
                                try:
#                                    ActionChains(driver).move_to_element(upload_label).click().perform()
                                    print("Сработал метод: ActionChains click на upload-label")
                                except Exception as ac_e:
                                    print("Метод ActionChains click не удался:", ac_e)
                        except Exception as e:
                            print("Не удалось найти элемент upload-photo:", e)

                        # Выбор из выпадающего списка "Кишинёв мун." с проверкой
                        try:
                            select_elem = WebDriverWait(driver, 10).until(
                                EC.presence_of_element_located((By.CSS_SELECTOR, "select.style_select__input__h0wAV[name='#7.value']"))
                            )
                            dropdown = Select(select_elem)
#                            dropdown.select_by_visible_text("Кишинёв мун.")
                            print("Выбран пункт 'Кишинёв мун.' из выпадающего списка")
                        except Exception as e:
                            print("Ошибка при выборе из выпадающего списка:", e)
                        # Попытка 1: Click через JavaScript
                        # Ждем поле для названия и вставляем название
                        # Попытка 2: Click через ActionChains
                        try:
                            upload_label_ac = WebDriverWait(driver, 10).until(
                                EC.element_to_be_clickable((By.CSS_SELECTOR, "label[for='upload-photo']"))
                            )
#                            ActionChains(driver).move_to_element(upload_label_ac).click().perform()
                            print("Сработал метод: ActionChains click на upload-label")
                        except Exception as e:
                            print("Метод ActionChains click не удался:", e)
                else:
                    print(f"Кнопка найдена?")
                    return True


            except Exception:
                try:

                    profile_url = "https://999.md/ru/profile/EgorCeban"
                    driver.get(profile_url)
                    pag.sleep(5)
                    location = pag.locateOnScreen(Wash, confidence=0.99)
                    pag.sleep(1)
                    if location is not None:
                        print(f"Кнопка Wash найдена")
                        # Переход к странице добавления объявления на Marketplace
                        url = "https://www.facebook.com/marketplace/create/item"
                        driver.get(url)
                        pag.sleep(1)

                        # Ждем поле для названия и вставляем название
                        WebDriverWait(driver, 20).until(
                            EC.presence_of_element_located((By.CSS_SELECTOR, "input.x1i10hfl.xggy1nq.xtpw4lu.x1tutvks.x1s3xk63.x1s07b3s.x1kdt53j.x1a2a7pz.xjbqb8w.x1ejq31n.xd10rxx.x1sy0etr.x17r0tee.x9f619.xzsf02u.x1uxerd5.x1fcty0u.x132q4wb.x1a8lsjc.x1pi30zi.x1swvt13.x9desvi.xh8yej3"))
                        )
                        title_input = driver.find_element(By.CSS_SELECTOR, "input.x1i10hfl.xggy1nq.xtpw4lu.x1tutvks.x1s3xk63.x1s07b3s.x1kdt53j.x1a2a7pz.xjbqb8w.x1ejq31n.xd10rxx.x1sy0etr.x17r0tee.x9f619.xzsf02u.x1uxerd5.x1fcty0u.x132q4wb.x1a8lsjc.x1pi30zi.x1swvt13.x9desvi.xh8yej3")
                        title_input.send_keys(title_text)
                        print("Название вставлено в поле ввода.")

                        pag.sleep(1)

                        button = WebDriverWait(driver, 30).until(
                            EC.element_to_be_clickable((By.XPATH, "//span[text()='Дополнительная информация']/ancestor::div[@role='button']"))
                        )
                        print("Текст кнопки:", button.text)
                        button.click()
                        print("Нажата кнопка: 'Дополнительная информация'")
                        try:
                            desc_input_locator = (By.ID, "«rfp»")
                            desc_input_field = WebDriverWait(driver, 50).until(
                                EC.element_to_be_clickable(desc_input_locator)
                            )
                            print("Поле описания найдено и кликабельно.")

                            # Шаг 3: Вставка текста в поле описания
                            desc_input_field.send_keys(description_text)
                            print(f"Текст '{description_text}' вставлен в поле описания.")

                        except Exception as e:
                            print(f"Ошибка при взаимодействии с полем описания: {e}")
                            # Здесь можно добавить логику для повторной попытки или завершения скрипта
                        print("Текст поля описания:", desc_input.get_attribute("aria-label") or desc_input.get_attribute("placeholder") or desc_input.get_attribute("id"))
                        desc_input.click()
                        pag.press('enter')
                        print("Фокус установлен на поле описания через TAB и ENTER. Вставляем текст через pag...")

                        pyperclip.copy(description_text)
                        pag.sleep(0.5)
                        pag.hotkey('command', 'v')
                        print("Описание вставлено в поле через буфер обмена и pag.")

                        # Кишинёв мун.(pag.sleep(0.1),
                        pag.keyDown('shift')
                        pag.press('tab', presses=1)
                        pag.keyUp('shift')
                        pag.press('down', presses=19, interval=0.01)
                        pag.press('enter')
                        # MDL
                        (pag.sleep(1), pag.press('tab', presses=2, interval=0.01),
                         pag.press('down', presses=3, interval=0.01), pag.press('enter'))

                        (pag.sleep(1), pag.press('tab', presses=2, interval=0.01),
                         pag.press('down', presses=2, interval=0.01), pag.press('enter'))  # Изменено на 2

                        (pag.sleep(1), pag.keyDown('shift'),
                         pag.press('tab', presses=2), pag.keyUp('shift'),
                         pag.sleep(1), pag.press('space'))
                        (pag.sleep(1),
                         pag.press('tab', presses=3), pag.keyDown('shift'),
                         pag.press('tab', presses=2), pag.keyUp('shift'),
                         pag.sleep(1), pag.press('space'))
                        (pag.sleep(2),
                         pag.click(), pag.sleep(2),
                         pag.press('down', presses=1, interval=0.01), pag.press('down', presses=1, interval=0.01),
                         pag.keyDown('shift'), pag.press('up', presses=1, interval=0.01),
                         pag.keyUp('shift'), pag.press('enter'))
                        pag.sleep(2)
                        (pag.sleep(1), pag.press('tab', presses=9),
                         pag.keyDown('shift'), pag.press('tab', presses=1),
                         pag.keyUp('shift'), pag.sleep(0.1),
                         pag.press('space'))
                        (pag.sleep(0.1), pag.press('tab', presses=3),
                         pag.sleep(0.1), pag.press('space'))
                        conf = pag.confirm("Продолжить?", "Confirmation")

                        # === Micro ===
                        location = pag.locateOnScreen(Micro, confidence=0.11)
                        pag.sleep(1)
                        if location is not None:
                            print(f"Кнопка Micro найдена")
                            url = "https://999.md/ru/add?category=household-appliances&subcategory=household-appliances%2Fmicrowaves"
                            driver.get(url)
                            pag.sleep(1)
                            WebDriverWait(driver, 20).until(
                                EC.presence_of_element_located((By.CSS_SELECTOR, "textarea[name='#13.value']"))
                            )
                            title_input = driver.find_element(By.CSS_SELECTOR, "input[name='#12.value.ru']")
                            title_input.send_keys(title_text)
                            print("Название вставлено в поле ввода.")
                            pag.sleep(1)
                            description_input = driver.find_element(By.CSS_SELECTOR, "textarea[name='#13.value']")
                            description_input.send_keys(description_text)
                            print("Описание вставлено в поле ввода.")
                            price_input = driver.find_element(By.CSS_SELECTOR, "input[name='#2.value.value']")
                            price_input.send_keys(price_text)
                            print("Цена вставлена в поле ввода.")
                            # ...дальнейшие действия по заполнению формы, как выше...
                            conf = pag.confirm("Продолжить?", "Confirmation")
                        else:
                            print(f"Кнопка Micro не найдена.")

                        # === Dish ===
                        location = pag.locateOnScreen(Dish, confidence=0.11)
                        pag.sleep(1)
                        if location is not None:
                            print(f"Кнопка Dish найдена")
                            url = "https://999.md/ru/add?category=household-appliances&subcategory=household-appliances%2Fdishwashers"
                            driver.get(url)
                            pag.sleep(1)
                            WebDriverWait(driver, 20).until(
                                EC.presence_of_element_located((By.CSS_SELECTOR, "textarea[name='#13.value']"))
                            )
                            title_input = driver.find_element(By.CSS_SELECTOR, "input[name='#12.value.ru']")
                            title_input.send_keys(title_text)
                            print("Название вставлено в поле ввода.")
                            pag.sleep(1)
                            description_input = driver.find_element(By.CSS_SELECTOR, "textarea[name='#13.value']")
                            description_input.send_keys(description_text)
                            print("Описание вставлено в поле ввода.")
                            price_input = driver.find_element(By.CSS_SELECTOR, "input[name='#2.value.value']")
                            price_input.send_keys(price_text)
                            print("Цена вставлена в поле ввода.")
                            # ...дальнейшие действия по заполнению формы, как выше...
                            conf = pag.confirm("Продолжить?", "Confirmation")
                        else:
                            print(f"Кнопка Dish не найдена.")

                        # === Coffee ===
                        location = pag.locateOnScreen(Coffee, confidence=0.11)
                        pag.sleep(1)
                        if location is not None:
                            print(f"Кнопка Coffee найдена")
                            url = "https://999.md/ru/add?category=household-appliances&subcategory=household-appliances/coffee-machines"
                            driver.get(url)
                            pag.sleep(1)
                            WebDriverWait(driver, 20).until(
                                EC.presence_of_element_located((By.CSS_SELECTOR, "textarea[name='#13.value']"))
                            )
                            title_input = driver.find_element(By.CSS_SELECTOR, "input[name='#12.value.ru']")
                            title_input.send_keys(title_text)
                            print("Название вставлено в поле ввода.")
                            pag.sleep(1)
                            description_input = driver.find_element(By.CSS_SELECTOR, "textarea[name='#13.value']")
                            description_input.send_keys(description_text)
                            print("Описание вставлено в поле ввода.")
                            price_input = driver.find_element(By.CSS_SELECTOR, "input[name='#2.value.value']")
                            price_input.send_keys(price_text)
                            print("Цена вставлена в поле ввода.")
                            # ...дальнейшие действия по заполнению формы, как выше...
                            conf = pag.confirm("Продолжить?", "Confirmation")
                        else:
                            print(f"Кнопка Coffee не найдена.")
                except Exception as error:
                    print(f"Произошла ошибка: {error}")
                    continue

if __name__ == "__main__":
    main()
