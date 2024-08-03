import requests
from bs4 import BeautifulSoup

# Step 1: レシピを検索し、食材をリストアップ
def get_recipe_ingredients(url):
    response = requests.get(url)
    soup = BeautifulSoup(response.content, 'html.parser')
    
    # 例としてクックパッドのレシピから食材を抽出
    ingredients = [item.text.strip() for item in soup.select('.ingredient_name')]
    return ingredients

# Step 2: 食品成分データベースを使用して食材の栄養情報を取得
def get_nutritional_info(ingredient):
    search_url = f'https://fooddb.mext.go.jp/index.pl?SEARCH_WORD={ingredient}'
    response = requests.get(search_url)
    soup = BeautifulSoup(response.content, 'html.parser')

    # 検索結果ページから詳細ページへのリンクを取得
    detail_link = soup.select_one('a[href*="food_detail"]')
    if detail_link:
        detail_url = 'https://fooddb.mext.go.jp/' + detail_link['href']
        detail_response = requests.get(detail_url)
        detail_soup = BeautifulSoup(detail_response.content, 'html.parser')

        # 栄養情報を取得
        nutrition = {
            'calories': 0,
            'protein': 0,
            'fat': 0,
            'carbs': 0
        }
        
        # 各栄養素の値を抽出
        nutrients = detail_soup.select('.data_box tr')
        for nutrient in nutrients:
            name = nutrient.select_one('th').text.strip()
            value = nutrient.select_one('td').text.strip()
            if name == 'エネルギー (kcal)':
                nutrition['calories'] = float(value)
            elif name == 'タンパク質 (g)':
                nutrition['protein'] = float(value)
            elif name == '脂質 (g)':
                nutrition['fat'] = float(value)
            elif name == '炭水化物 (g)':
                nutrition['carbs'] = float(value)
        
        return nutrition
    else:
        return {
            'calories': 0,
            'protein': 0,
            'fat': 0,
            'carbs': 0
        }

# Step 3: 食材ごとの栄養情報を合計
def calculate_total_nutrition(ingredients):
    total_nutrition = {
        'calories': 0,
        'protein': 0,
        'fat': 0,
        'carbs': 0
    }
    
    for ingredient in ingredients:
        nutrition = get_nutritional_info(ingredient)
        for key in total_nutrition:
            total_nutrition[key] += nutrition[key]
    
    return total_nutrition

# 使用するレシピのURL
recipe_url = 'https://cookpad.com/recipe/1234567'  # 実際のURLに置き換えてください

# 食材リストの取得
ingredients = get_recipe_ingredients(recipe_url)
print("食材:", ingredients)

# 栄養情報の計算
total_nutrition = calculate_total_nutrition(ingredients)
print("栄養情報の合計:", total_nutrition)
