# Gift Optimization

## Methodology

The goal for the gift optimization analysis is to take every Stardew Valley Villager’s Likes and Loved gifts and make a cluster model that will group the most common liked gifts. From there then we can optimize or gift giving by producing those specific gifts and bring our hearts up in the least amount of time. 

## Data Source and cleaning

The source for this section if the Stardew Valley wiki page. I scraped all the tables in the page using pandas and proceeded to clean them.

The data cleaning process took some time since not only did I join all tables, but also the format on which the tables were scraped had multiple items on each cell I had to separate using the explode() command on pandas. After exploding every single item I had a giant dataset with every single item repeated multiple times for each column except the last one (last to explode).

After this, I decided it was better to deal with the data having the Name, Taste (Likes, Loves, Hates, etc), and item as column names. So I made the appropriate transformations and ended up with the final dataset I will be using for this analysis.

 

## Image Sourcing

I wanted to illustrate my results by having each item’s name at the top of the bar charts, so I had to scrape the images using BeautifulSoup on the Stardew Valleys Wiki (pls dont block me Concerned Ape).

```python
Scrape images
item_list = counts_sorted['Item'].unique()
item_urls = []

 for item in item_list:
     print("Extracting: ", item)
     try:
        item_url = f'https://stardewvalleywiki.com/{item}'
        item_html = requests.get(item_url).content
        soup = BeautifulSoup(item_html,'html.parser')
        box = soup.find(id='infoboxtable')
        image = box.find_next('img')['src']
        item_urls.append((item, f'https://stardewvalleywiki.com{image}'))
     except Exception as e:
         print(e)
         item_urls.append((item, ""))

image_df = pd.DataFrame(item_urls, columns=['Item', 'url'])
image_df.to_csv('/Users/myname/Documents/Stardew Valley Project/ItemImages.csv')
```

After having these images saved, I created a bar plot to illustrate which items are the most loved.

```python
labels = joined['Item']
values = joined['Count']

# Set the height of the bars
height = 0.9

# Create the horizontal bar chart
plt.barh(y=labels, width=values, height=height, color='skyblue', align='center')

# Add images to the end of each bar
for i, row in joined.iterrows():
    # Fetch the image from URL
    response = requests.get(row["url"])
    img = plt.imread(BytesIO(response.content))
    
    # Determine the position for the image
    img_x = row["Count"]  # Image x-coordinate (end of the bar)
    img_y = i  # Image y-coordinate (centered on the bar)
    img_width = height * 0.7  # Set the width of the image to match the bar height
    
    # Display the image
    plt.imshow(img, extent=[img_x - img_width, img_x, img_y - height / 2, img_y + height / 2], aspect='auto', zorder=2)

# Set plot limits and labels
plt.xlim(0, max(values) * 1.05)  # Set x-axis limits
plt.ylim(-0.5, len(labels) - 0.5)  # Set y-axis limits
plt.tight_layout()
plt.show(
```



## Optimizing gifts for quick friendships

Since my interest is to be able to form relationships more efficiently, I focused on the loved items in the game. Since there are many NPCs that love the same items, I figured a way to group them so that I have to focus on fewer items to obtain and be able to gift them in bulk.

For this, I iterated over the list of each most liked item, and assigned each NPC to the most popular item they love. For example, if Clint likes Amethyst and Carrots, Amethyst is liked by other 3 NPCs while carrots are only loved by Clint, so Clint is assigned to the Amethyst group and no other group. 

![Untitled](https://github.com/daniellemontalvo/Stardew-Valley-Optimization/assets/81642044/815118d8-5d39-44c9-9dac-8cfd5f3e8007)
![Untitled 2](https://github.com/daniellemontalvo/Stardew-Valley-Optimization/assets/81642044/21a92510-17f2-49a3-9695-f137ab719101)
![Untitled 1](https://github.com/daniellemontalvo/Stardew-Valley-Optimization/assets/81642044/aea276fa-b457-44f7-9b03-6548bc1257ef)


```python

# Initialize a dictionary to keep track of villagers assigned to items
assigned_villagers = {}

# Iterate over each row in the DataFrame
for index, row in loved_items.iterrows():
    item = row['Item']
    villagers = row['Names'].split(', ')
    new_villagers = []
    
    # Check each villager in the current row
    for villager in villagers:
        # If the villager has not been assigned to any item yet, add them to the current item's row
        if villager not in assigned_villagers:
            assigned_villagers[villager] = item
            new_villagers.append(villager)

    # Update the 'Names' column with only the new villagers
    loved_items.at[index, 'Names'] = ', '.join(new_villagers)
loved_items = loved_items[loved_items['Names'].str.strip().astype(bool)]

# Print the updated DataFrame
loved_items
```

<img width="541" alt="Untitled 3" src="https://github.com/daniellemontalvo/Stardew-Valley-Optimization/assets/81642044/654c980c-118f-4ac2-aad0-59ea8f22b54f">


From this list, we can see that most NPCs fall into the Diamond group (because who doesn’t love diamonds). But also, there are many NPCs that are alone in their group. Which is why I thought about making  big group that will belong to the ‘Golden Pumpkin’ group since it is a Universal loved item. However, this was not the best way to optimize it, since the odds of getting Golden Pumpkins in the game are slim. 

From the Universal loves list, I found that ‘Rabbits Foot’ is the easiest one to get. This item however, is not loved by Penny, but it is fine since Penny has already been assigned to the Diamonds group.

After the necessary adjustments this is what we get:

 

<img width="888" alt="Untitled 4" src="https://github.com/daniellemontalvo/Stardew-Valley-Optimization/assets/81642044/c2bc6ca9-383a-4129-be9e-5eb134241439">


# Dash Display

I wanted to explore some of the possibilities for illustrating this data. After some conversations with chat GPT I decided to make a Dash app that would display the item names and in a pop up say the list of NPCs who like said item. I am still iterating over this dashboard but this is the first draft:

<img width="1436" alt="Untitled 5" src="https://github.com/daniellemontalvo/Stardew-Valley-Optimization/assets/81642044/5c0dc685-0e4e-4e38-a521-a54ef0786b7e">


After a few iterations and me playing around with Dash, this is the final product:

![Untitled 6](https://github.com/daniellemontalvo/Stardew-Valley-Optimization/assets/81642044/8e995b82-8c11-4f8a-b882-21af67893643)


There are many ways I could change this dashboard to make it look better, but I am more interested in the analytics side, so I stopped here and moved on to the next analytic part I was interested in.
