# Google Apps Script 
# Weekly Meal Plan Generator

## **Project Links**
- [Google Apps Script Project](https://script.google.com/home/projects/1gddqAKb4sovmMAw4B01FEIpo2ZpkgiQaIQh_0OUj1dI_jxjzu_PgXfSa/edit)
- [Google Spreadsheet Template](https://docs.google.com/spreadsheets/d/1r5IqbryBqSTwhdI23hpI2UXmj6A7zvVXbBnpNoLHd7Q/edit?gid=760759012#gid=760759012)

---

## **Overview**
This project is a Google Apps Script that helps users generate a weekly meal plan based on their dietary preferences, calorie goals, and excluded ingredients. It uses the Spoonacular API to fetch meal plans and organizes the information in a Google Sheet.

---

## **Features**
1. **User Preferences Input**: Users can specify their calorie goals, diet type, excluded ingredients, and the number of days for the plan.
2. **API Integration**: The script connects to the Spoonacular API to fetch meal plans tailored to the user's preferences.
3. **Dynamic Sheet Updates**: Automatically clears and formats the sheet to display a detailed weekly meal plan, including:
   - Meal titles
   - Recipe links
   - Nutritional information (calories, fat, protein, carbohydrates)
4. **Customizable and Scalable**: Easy to modify for additional preferences or advanced meal planning.

---

## **How It Works**
### **1. User Preferences**
The user enters preferences in the "User Input" sheet:
- **Calorie Goal**: Target calorie intake per day.
- **Diet**: Dietary preference (e.g., vegan, ketogenic, etc.).
- **Excluded Ingredients**: Ingredients to avoid (comma-separated).
- **Number of Days**: Number of days for the meal plan.

The script fetches these preferences using the `getUserPreferences` function.

### **2. Generate Meal Plan**
The `generateMealPlan` function builds a request to the Spoonacular API using the user’s preferences:
- `calorieGoal`
- `diet`
- `excludeIngredients`
- `numberOfDays`

It retrieves data, including meals for each day and their nutritional details, and returns the meal plan.

### **3. Display Meal Plan**
The `displayMealPlan` function:
1. Clears the "Weekly Meal Plan" sheet.
2. Formats headers and preference details.
3. Populates the sheet with:
   - Day-wise meals
   - Recipe links (with clickable URLs)
   - Nutritional information (calories, fat, protein, carbs).

---

## **Key Code Functions**

### **1. getUserPreferences**
Reads user preferences from the "User Input" sheet.
```javascript
function getUserPreferences() {
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('User Input');
  const preferences = {
    calorieGoal: sheet.getRange('A2').getValue(),
    diet: sheet.getRange('B2').getValue(),
    excludeIngredients: sheet.getRange('C2').getValue().split(',').map(e => e.trim()),
    numberOfDays: sheet.getRange('D2').getValue()
  };
  Logger.log(preferences);
  return preferences;
}
```

### **2. generateMealPlan**
Fetches a meal plan from the Spoonacular API.
```javascript
function generateMealPlan() {
  const preferences = getUserPreferences();
  const timeFrame = preferences.numberOfDays >= 7 ? 'week' : 'day';
  const endpoint = `${SPOONACULAR_API_BASE_URL}/mealplanner/generate?` + 
                   `timeFrame=${timeFrame}` + 
                   `&targetCalories=${preferences.calorieGoal}` + 
                   `&diet=${encodeURIComponent(preferences.diet)}` + 
                   `&exclude=${encodeURIComponent(preferences.excludeIngredients.join(','))}` + 
                   `&apiKey=${SPOONACULAR_API_KEY}`;
  try {
    const response = UrlFetchApp.fetch(endpoint);
    const mealPlanData = JSON.parse(response.getContentText());
    Logger.log(mealPlanData);
    return mealPlanData;
  } catch (error) {
    Logger.log('Error fetching meal plan: ' + error.message);
    return null;
  }
}
```

### **3. displayMealPlan**
Populates the "Weekly Meal Plan" sheet with meal plan details.
```javascript
function displayMealPlan() {
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('Weekly Meal Plan');
  const preferences = getUserPreferences();
  const mealPlanData = generateMealPlan();
  if (!mealPlanData || !mealPlanData.week) {
    Logger.log("Meal plan data is empty or invalid.");
    return;
  }

  sheet.clear();
  sheet.clearFormats();

  sheet.getRange('A1').setValue('Weekly meals plan').setFontWeight('bold').setFontSize(14).setHorizontalAlignment('center');
  sheet.getRange('A1:B1').merge();
  sheet.getRange('A2').setValue('Calorie Goal').setFontWeight('bold').setFontColor('black');
  sheet.getRange('B2:D2').merge().setValue(preferences.calorieGoal).setFontColor('black').setBackground('white').setHorizontalAlignment('right');

  let startRow = 8;
  sheet.getRange(startRow, 1, 1, 7).setValues([['Day', 'Meal', 'Recipe Link', 'Calories', 'Fat (g)', 'Protein (g)', 'Carbohydrates (g)']])
                                    .setFontWeight('bold').setFontSize(12).setBackground('#4caf50').setFontColor('white');
  let row = startRow + 1;

  for (const [day, dayData] of Object.entries(mealPlanData.week)) {
    const nutrients = dayData.nutrients;
    sheet.getRange(row, 1).setValue(day.charAt(0).toUpperCase() + day.slice(1)).setFontWeight('bold').setBackground('#d4edbc').setFontColor('black');

    dayData.meals.forEach((meal, index) => {
      if (index > 0) row++;
      sheet.getRange(row, 2).setValue(meal.title).setFontColor('black');
      sheet.getRange(row, 3).setFormula(`=HYPERLINK("${meal.sourceUrl}", "View Recipe")`).setFontColor('black');
    });

    row++;
  }
  Logger.log("Meal plan displayed in the sheet.");
}
```

---

## **Setup Instructions**
1. Open a Google Sheet and go to **Extensions > Apps Script**.
2. Paste the code above into the Apps Script editor.
3. Replace `SPOONACULAR_API_KEY` with your API key.
4. Create two sheets:
   - **User Input**: For user preferences (calorie goal, diet, etc.).
   - **Weekly Meal Plan**: To display the meal plan.
5. Save and run the `displayMealPlan` function.

---

## **Technologies Used**
- **Google Apps Script**: For automation and sheet manipulation.
- **Spoonacular API**: For meal planning data.
- **Google Sheets**: For user input and output display.



