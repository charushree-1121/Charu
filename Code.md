# Charu
YouTube Data Harvesting and Warehousing using SQL, MongoDB and Streamlit
import streamlit as st
import pandas as pd
from pymongo import MongoClient
import seaborn as sns
import matplotlib.pyplot as plt

# Connect to MongoDB
client = MongoClient('mongodb://localhost:27017/')
db = client['youtube_data']
video_collection = db['video']
channel_collection = db['channel']
comment_collection = db['comment']



# Function to fetch all video data
def fetch_video_data():
    return list(video_collection.find({}, {"_id": 0}))

# Function to fetch all channel data
def fetch_channel_data():
    return list(channel_collection.find({}, {"_id": 0}))

# Function to fetch all comment data
def fetch_comment_data():
    return list(comment_collection.find({}, {"_id": 0}))

# Function to display names of all videos and their corresponding channels
def display_videos_for_selected_channel(video_data, channel_data):
    # Create a channel selector dropdown
    selected_channel = st.sidebar.selectbox("Select a Channel", ['All Channels'] + [channel['Channel_Name'] for channel in channel_data])

    # Display videos based on the selected channel
    if selected_channel == 'All Channels':
        st.subheader('All Videos:')
        for video in video_data:
            video_name = video.get('Video_Name', 'N/A')
            video_channel_id = video.get('Channel_Id', '')

            # Find the corresponding channel name
            channel_name = next((channel.get('Channel_Name', 'N/A') for channel in channel_data if channel.get('Channel_Id') == video_channel_id), 'N/A')

            st.write(f"Video Name: {video_name} - Channel Name: {channel_name}")
    else:
        st.subheader(f'Videos of {selected_channel}:')
        for video in video_data:
            video_name = video.get('Video_Name', 'N/A')
            video_channel_id = video.get('Channel_Id', '')

            # Find the corresponding channel name
            channel_name = next((channel.get('Channel_Name', 'N/A') for channel in channel_data if channel.get('Channel_Id') == video_channel_id), 'N/A')

            if channel_name == selected_channel:
                st.write(f"Video Name: {video_name} - Channel Name: {channel_name}")


def display_top_10_most_viewed_videos(video_data, channel_data):
    # Convert view counts to integers for sorting
    for video in video_data:
        view_count_str = str(video.get('View_Count', '0'))  # Ensure it's a string
        try:
            video['View_Count'] = int(view_count_str.replace(',', ''))  # Convert and remove commas
        except ValueError:
            # Handle cases where view count is not a valid integer (e.g., 'No views')
            video['View_Count'] = 0

    # Sort videos by view count in descending order
    sorted_videos = sorted(video_data, key=lambda x: x.get('View_Count', 0), reverse=True)

    st.subheader('Top 10 Most Viewed Videos:')
    for i, video in enumerate(sorted_videos[:10], start=1):
        video_name = video.get('Video_Name', 'N/A')
        video_channel_id = video.get('Channel_Id', '')
        view_count = video.get('View_Count', 0)

        # Find the corresponding channel name
        channel_name = next((channel.get('Channel_Name', 'N/A') for channel in channel_data if channel.get('Channel_Id') == video_channel_id), 'N/A')

        st.write(f"{i}. Video Name: {video_name} - Channel Name: {channel_name} - View Count: {view_count}")

    # Create a Seaborn bar chart to visualize the top 10 most viewed videos
    top_10_video_names = [video.get('Video_Name', 'N/A') for video in sorted_videos[:10]]
    top_10_view_counts = [video.get('View_Count', 0) for video in sorted_videos[:10]]

    fig, ax = plt.subplots(figsize=(10, 6))
    sns.barplot(x=top_10_view_counts, y=top_10_video_names, palette="viridis")
    plt.xlabel('View Count')
    plt.ylabel('Video Name')
    plt.title('Top 10 Most Viewed Videos')

    st.pyplot(fig)

    # Create a Seaborn pie chart to show the distribution of views among the top 10 videos
    fig, ax = plt.subplots()
    ax.pie(top_10_view_counts, labels=top_10_video_names, autopct='%1.1f%%', startangle=90)
    ax.axis('equal')  # Equal aspect ratio ensures that pie is drawn as a circle.
    plt.title('Distribution of Views Among Top 10 Videos')

    st.pyplot(fig)

# Function to display the top 10 most viewed videos and their respective channels
def dummy_display_top_10_most_viewed_videos(video_data, channel_data):
    # Convert view counts to integers for sorting
    for video in video_data:
        view_count_str = str(video.get('View_Count', '0'))  # Ensure it's a string
        try:
            video['View_Count'] = int(view_count_str.replace(',', ''))  # Convert and remove commas
        except ValueError:
            # Handle cases where view count is not a valid integer (e.g., 'No views')
            video['View_Count'] = 0

    # Sort videos by view count in descending order
    sorted_videos = sorted(video_data, key=lambda x: x.get('View_Count', 0), reverse=True)

    st.subheader('Top 10 Most Viewed Videos:')
    for i, video in enumerate(sorted_videos[:10], start=1):
        video_name = video.get('Video_Name', 'N/A')
        video_channel_id = video.get('Channel_Id', '')
        view_count = video.get('View_Count', 0)

        # Find the corresponding channel name
        channel_name = next((channel.get('Channel_Name', 'N/A') for channel in channel_data if channel.get('Channel_Id') == video_channel_id), 'N/A')

        st.write(f"{i}. Video Name: {video_name} - Channel Name: {channel_name} - View Count: {view_count}")

# Function to find channels with the most number of videos
def channels_with_most_videos(video_data, channel_data):
    channel_video_counts = {}

    # Calculate the number of videos per channel
    for video in video_data:
        channel_id = video.get('Channel_Id', '')
        if channel_id in channel_video_counts:
            channel_video_counts[channel_id] += 1
        else:
            channel_video_counts[channel_id] = 1

    # Sort channels by the number of videos
    sorted_channels = sorted(channel_video_counts.items(), key=lambda x: x[1], reverse=True)

    st.subheader('Channels with the Most Number of Videos:')
    for channel_id, video_count in sorted_channels[:5]:  # Display the top 5 channels
        # Find the corresponding channel name
        channel_name = next((channel.get('Channel_Name', 'N/A') for channel in channel_data if channel.get('Channel_Id') == channel_id), 'N/A')
        st.write(f"Channel Name: {channel_name} - Number of Videos: {video_count}")

# Function to display videos with the highest number of likes and their corresponding channel names
def display_videos_with_highest_likes(video_data, channel_data):
    # Convert like counts to integers for sorting
    for video in video_data:
        like_count_str = str(video.get('Like_Count', '0'))  # Ensure it's a string
        try:
            video['Like_Count'] = int(like_count_str.replace(',', ''))  # Convert and remove commas
        except ValueError:
            # Handle cases where like count is not a valid integer
            video['Like_Count'] = 0

    # Sort videos by like count in descending order
    sorted_videos = sorted(video_data, key=lambda x: x.get('Like_Count', 0), reverse=True)

    st.subheader('Videos with the Highest Likes:')
    for i, video in enumerate(sorted_videos[:10], start=1):
        video_name = video.get('Video_Name', 'N/A')
        video_channel_id = video.get('Channel_Id', '')
        like_count = video.get('Like_Count', 0)

        # Find the corresponding channel name
        channel_name = next((channel.get('Channel_Name', 'N/A') for channel in channel_data if channel.get('Channel_Id') == video_channel_id), 'N/A')

        st.write(f"{i}. Video Name: {video_name} - Channel Name: {channel_name} - Like Count: {like_count}")

# Function to calculate the total views per channel
def calculate_total_views_per_channel(video_data):
    total_views_per_channel = {}

    for video in video_data:
        channel_id = video.get('Channel_Id', '')
        view_count_str = str(video.get('View_Count', '0'))  # Ensure it's a string
        try:
            view_count = int(view_count_str.replace(',', ''))  # Convert and remove commas
        except ValueError:
            # Handle cases where view count is not a valid integer (e.g., 'No views')
            view_count = 0

        if channel_id in total_views_per_channel:
            total_views_per_channel[channel_id] += view_count
        else:
            total_views_per_channel[channel_id] = view_count

    return total_views_per_channel

# Function to display the total views per channel
def display_total_views_per_channel(video_data, channel_data):
    st.subheader('Total Views per Channel:')
    
    # Calculate the total views per channel
    total_views_per_channel = calculate_total_views_per_channel(video_data)

    for channel in channel_data:
        channel_id = channel.get('Channel_Id', '')
        channel_name = channel.get('Channel_Name', 'N/A')

        total_views = total_views_per_channel.get(channel_id, 0)
        st.write(f"Channel Name: {channel_name} - Total Views: {total_views}")

# Function to display the total likes and dislikes for each video
def display_total_likes_and_dislikes(video_data):
    st.subheader('Total Likes and Dislikes for Each Video:')
    for video in video_data:
        video_name = video.get('Video_Name', 'N/A')
        like_count = video.get('Like_Count', 0)
        dislike_count = video.get('Dislike_Count', 0)

        st.write(f"Video Name: {video_name} - Total Likes: {like_count} - Total Dislikes: {dislike_count}")

# Function to display channels that published videos in a specific year
def display_channels_published_in_year(video_data, channel_data, year):
    st.subheader(f'Channels that Published Videos in {year}:')
    
    # Extract the year from the published date of each video
    channels_published_in_year = set()
    for video in video_data:
        published_date = video.get('PublishedAt', '')
        if year in published_date:
            channel_id = video.get('Channel_Id', '')
            channels_published_in_year.add(channel_id)

    # Find and display the channel names
    for channel in channel_data:
        channel_id = channel.get('Channel_Id', '')
        if channel_id in channels_published_in_year:
            channel_name = channel.get('Channel_Name', 'N/A')
            st.write(f"Channel Name: {channel_name}")



def display_average_duration_per_channel(video_data, channel_data):
    # Convert video_data and channel_data lists into DataFrames
    video_df = pd.DataFrame(video_data)
    channel_df = pd.DataFrame(channel_data)

    # Check if 'channel_name' is a valid column in video_df
    if 'Channel_Id' in video_df.columns:
        # Group video_df by 'channel_name' and calculate the average duration
        average_duration_per_channel = video_df.groupby('Channel_Id')['Duration'].apply(lambda x: pd.to_timedelta(x).mean()).reset_index()

        # Convert the average duration to MM:SS format
        average_duration_per_channel['Average Duration (MM:SS)'] = average_duration_per_channel['Duration'].apply(lambda x: str(x).split()[-1])

        # Sort the results by duration (descending)
        average_duration_per_channel = average_duration_per_channel.sort_values(by='Duration', ascending=False).reset_index(drop=True)

        # Display the result using Streamlit
        st.dataframe(average_duration_per_channel[['Channel_Id', 'Average Duration (MM:SS)']])
   

# Example data
#video_data = [...]  # List of dictionaries containing video data
#channel_data = [...]  # List of dictionaries containing channel data

# Call the function
#display_average_duration_per_channel(video_data, channel_data)


# Function to find videos with the highest number of comments
def find_videos_with_highest_comments(video_data, comment_data):
    video_comments_count = {}

    for comment in comment_data:
        video_id = comment.get('video_id', '')
        if video_id in video_comments_count:
            video_comments_count[video_id] += 1
        else:
            video_comments_count[video_id] = 1

    # Sort videos by the number of comments
    sorted_videos = sorted(video_comments_count.items(), key=lambda x: x[1], reverse=True)

    return sorted_videos

# Function to display videos with the highest number of comments and their corresponding channel names
def display_videos_with_highest_comments(video_data, channel_data, comment_data):
    st.subheader('Videos with the Highest Number of Comments:')
    
    # Find videos with the highest number of comments
    sorted_videos = find_videos_with_highest_comments(video_data, comment_data)

    for i, (video_id, comments_count) in enumerate(sorted_videos[:10], start=1):
        video_name = next((video.get('Video_Name', 'N/A') for video in video_data if video.get('Video_Id') == video_id), 'N/A')
        video_channel_id = next((video.get('Channel_Id', '') for video in video_data if video.get('Video_Id') == video_id), '')

        # Find the corresponding channel name
        channel_name = next((channel.get('Channel_Name', 'N/A') for channel in channel_data if channel.get('Channel_Id') == video_channel_id), 'N/A')

        st.write(f"{i}. Video Name: {video_name} - Channel Name: {channel_name} - Number of Comments: {comments_count}")

# Function to find the number of comments for each video
def find_comments_per_video(comment_data):
    comments_per_video = {}

    for comment in comment_data:
        video_id = comment.get('video_id', '')
        if video_id in comments_per_video:
            comments_per_video[video_id].append(comment)
        else:
            comments_per_video[video_id] = [comment]

    return comments_per_video

# Function to display the number of comments for each video and their corresponding video names
def display_comments_per_video(video_data, comment_data):
    st.subheader('Number of Comments per Video:')
    
    # Find the number of comments for each video
    comments_per_video = find_comments_per_video(comment_data)

    for video in video_data:
        video_id = video.get('Video_Id', '')
        video_name = video.get('Video_Name', 'N/A')

        if video_id in comments_per_video:
            comments_list = comments_per_video[video_id]
            comments_count = len(comments_list)
            st.write(f"Video Name: {video_name} - Number of Comments: {comments_count}")

# Fetch data
video_data = fetch_video_data()
channel_data = fetch_channel_data()
comment_data = fetch_comment_data()


# Streamlit app
st.title('YouTube Data Analysis')

display_videos_button = st.sidebar.button("Display Videos for Selected Channel")
show_top_10_button = st.sidebar.button("Show Top 10 Most Viewed Videos")
show_channels_most_videos_button = st.sidebar.button("Show Channels with Most Videos")
show_videos_highest_likes_button = st.sidebar.button("Show Videos with Highest Likes")
show_total_views_button = st.sidebar.button("Show Total Views per Channel")
show_likes_dislikes_button = st.sidebar.button("Show Total Likes and Dislikes for Each Video")
show_channels_2022_button = st.sidebar.button("Show Channels Published Videos in 2022")
show_average_duration_button = st.sidebar.button("Show Average Duration of Videos in Each Channel")
show_highest_comments_button = st.sidebar.button("Show Videos with Highest Number of Comments")
show_comments_per_video_button = st.sidebar.button("Show Number of Comments per Video")


if display_videos_button:
    display_videos_for_selected_channel(video_data, channel_data)
if show_top_10_button:
    display_top_10_most_viewed_videos(video_data, channel_data)
if show_channels_most_videos_button:
    channels_with_most_videos(video_data, channel_data)
if show_videos_highest_likes_button:
    display_videos_with_highest_likes(video_data, channel_data)
if show_total_views_button:
    display_total_views_per_channel(video_data, channel_data)
if show_likes_dislikes_button:
    display_total_likes_and_dislikes(video_data)
if show_channels_2022_button:
    display_channels_published_in_year(video_data, channel_data, '2022')
if show_average_duration_button:
    display_average_duration_per_channel(video_data, channel_data)
if show_highest_comments_button:
    display_videos_with_highest_comments(video_data, channel_data, comment_data)
if show_comments_per_video_button:
    display_comments_per_video(video_data, comment_data)
