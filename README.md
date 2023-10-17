
# Pliot Study: Popularity factor analysis with Youtube data

# 서론

## Background

- 소셜 미디어는 브랜드 이미지 및 로열티 관리에 있어 중요한 역할을 수행(Bilgin, 2018)
- 소셜 미디어에는 많은 양의 정보가 있어 브랜드 관리를 목적으로 소셜 미디어 모니터링을 할 때 많은 시간과 비용이 필요(Zhang & Marita, 2014)
- 최근, 브랜드 관리와 같은 비즈니스 전략 수립을 목적으로 한 소셜 미디어 분석 연구가 활발히 이루어짐(Greco & Polli, 2020; Jai et al., 2022; Lim et al., 2020)

## Motivation

- 동일한 마케팅 전략이어도 브랜드 이미지와 고객 성향에 따라 성패가 달라짐(Dash et al., 2021)
- 브랜드에 따라 고객들의 성향을 파악하고 고객들이 중요시하는 가치를 알아내야 함(Bernarto et al., 2020)
- 소셜 미디어 모니터링으로 브랜드 고객의 성향을 알 수 있으나 단순 모니터링만으로는 마케터의 주관이 개입되어 분석 결과가 왜곡될 수 있음(Zhang & Marita, 2014)
- 소셜 미디어 컨텐츠 popularity의 변화를 설명하는 factor를 찾기 위해 토픽 모델링과 같은 머신러닝 기법으로 추출된 독립변수가 사용됨(Zhang et al., 2021)

## Purpose

- 브랜드 관련 유튜브 컨텐츠의 description 및 caption 데이터와 비디오의 메타 데이터를 활용하여 브랜드 고객들의 관심사와 선호에 대한 정보를 제공하고 그에 따른 마케팅 전략을 제시

# 데이터 수집

- 데이터 출처: Youtube Data API v3
    
    [API Reference  |  YouTube Data API  |  Google for Developers](https://developers.google.com/youtube/v3/docs?hl=ko)
    
- 데이터 수집 대상: 특정 브랜드 또는 제품과 관련된 유튜브 영상의 title, description, caption 데이터 (텍스트 데이터)
    - Apple iPhone 15
    - Samsung Galaxy S23
        
        → 특정 제품 또는 브랜드 단위로 실험을 진행할 예정이나, 이번 pilot study에서는 상기 두 제품을 case study로 진행
        

## 수집 절차

1. GCP(Google Cloud Platform)에서 Youtube Data API KEY 발급
2. Search 메소드를 사용하여 유튜브 동영상 검색 및 수집
    - 파라미터 목록
        1. query: 검색에 사용될 키워드. 본 실험에서는 “apple iphone 15”, “samsung galaxy s23”으로 설정
        2. order: API 응답에서 리소스를 정렬하는 데 사용할 메서드. 본 실험에서는 “relevance” 즉 관련도 순으로 설정
            - 가능한 다른 값
                - **`date`** – 리소스를 만든 날짜를 기준으로 최근 항목부터 시간 순서대로 리소스를 정렬합니다.
                - **`rating`** – 높은 평가부터 낮은 평가순으로 리소스를 정렬합니다.
                - **`relevance`** – 검색어와의 관련성을 기준으로 리소스를 정렬합니다. 이 매개변수의 기본값입니다.
                - **`title`** – 리소스가 제목순으로 알파벳순으로 정렬됩니다.
                - **`videoCount`** – 채널은 업로드한 동영상 수를 기준으로 내림차순으로 정렬됩니다.
                - **`viewCount`** – 리소스가 조회수가 높은 순으로 정렬됩니다. 실시간 방송의 경우 방송이 진행되는 동안 동시 시청자 수를 기준으로 동영상을 정렬합니다.
        3. part: search 메소드에서 수집 가능한 비디오 메타 데이터를 선택하는 파라미터. 본 실험에서는 “snippet”으로 설정
            - 참고) search API response의 구조
                
                ```
                {
                  "kind": "youtube#searchResult",
                  "etag":etag,
                  "id": {
                    "kind":string,
                    "videoId":string,
                    "channelId":string,
                    "playlistId":string
                  },
                  "snippet": {
                    "publishedAt":datetime,
                    "channelId":string,
                    "title":string,
                    "description":string,
                    "thumbnails": {
                (key): {
                        "url":string,
                        "width":unsigned integer,
                        "height":unsigned integer
                      }
                    },
                    "channelTitle":string,
                    "liveBroadcastContent":string
                  }
                }
                ```
                
        4. type: 특정 유형의 리소스만 검색하도록 검색어를 제한. 기본값은 “video, channel, playlist”로 모든 리소스를 포함하나 본 실험에서는 “video”로 설정
        5. videoCaption: 자막이 있는지에 따라 API가 동영상 검색결과를 필터링해야 하는지 여부. 본 실험에서는 자막이 있는 리소스만 포함하도록 “closedCaption”으로 설정
            - 가능한 다른 값
                - **`any`** - 자막 제공 여부에 따라 결과를 필터링하지 않습니다.
                - **`closedCaption`** - 자막이 있는 동영상만 포함합니다.
                - **`none`** - 자막이 없는 동영상만 포함합니다.
        6. maxResults: 결과 집합에 반환해야 하는 최대 항목 수를 지정. 최대 50까지 설정 가능하며, 기본값은 5. 본 실험에서는 50으로 설정.
    
3. search 메소드에서 가져올 수 없는 메타 데이터인 조회수를 수집하기 위해, 각 videoId를 Videos 메소드의 파라미터로 입력해 메타데이터 수집
4. Youtube Transcript API 라이브러리를 사용하여 모든 동영상의 자막 데이터 수집
    
    [youtube-transcript-api](https://pypi.org/project/youtube-transcript-api/)
    
    - 영어 자막인 경우 그대로 수집
    - 영어 자막이 아닌 경우 번역해서 수집
- Full code
    
    ```python
    def build_youtube_search(developer_key):
        DEVELOPER_KEY = developer_key
        YOUTUBE_API_SERVICE_NAME="youtube"
        YOUTUBE_API_VERSION="v3"
        return build(YOUTUBE_API_SERVICE_NAME,YOUTUBE_API_VERSION,developerKey=DEVELOPER_KEY)
    
    def my_search(api, query, page_token = ''):
        if not page_token:
            response = api.search().list(
                q = query,
                order = "relevance",
                part = "snippet",
                type = "video",
                videoCaption = "closedCaption",
                maxResults = 50,
                ).execute()
        else:
            response = api.search().list(
                q = query,
                order = "relevance",
                part = "snippet",
                type = "video",
                videoCaption = "closedCaption",
                maxResults = 50,
                pageToken = page_token
                ).execute()
        return response
    
    def get_search_response(youtube, query):
        search_items = []
        video_items = []
        INIT_REQ = True
        prev_ids = []
        i = 0
        while len(search_items) < 1000 and i < 8:
            if INIT_REQ:
                try:
                    search_response = my_search(youtube, query)
                    INIT_REQ = False
                except HttpError:
                    i += 1
                    if i >= 8:
                        break
                    try:
                        youtube = build_youtube_search(keys[i])
                        search_response = my_search(youtube, query)
                    except HttpError:
                        break
            else:
                try:
                    search_response = my_search(youtube, query, next_page_token)
                except HttpError:
                    i += 1
                    if i >= 8:
                        break
                    try:
                        youtube = build_youtube_search(keys[i])
                        search_response = my_search(youtube, query, next_page_token)
                    except HttpError:
                        break
                        
            search_items.extend(search_response['items'])
            
            new_results = []
            for s in search_items:
                if s not in new_results:
                    new_results.append(s)
            search_items = new_results
            
            video_ids = [i.get('id').get('videoId') for i in search_response['items'] if i['id']['kind'] == 'youtube#video']
            if not prev_ids:
                prev_ids = video_ids
                video_ids_new = video_ids
            else:
                video_ids_new = [vid for vid in video_ids if vid not in prev_ids]
                prev_ids += video_ids_new
            
            try:
                video_items.extend(youtube.videos().list(
                    part = "id,contentDetails,statistics",
                    id = ','.join(video_ids_new)).execute()['items'])
            except HttpError:
                i += 1
                if i >= 8:
                    break
                try:
                    youtube = build_youtube_search(keys[i])
                    video_items.extend(youtube.videos().list(
                        part = "id,contentDetails,statistics",
                        id = ','.join(video_ids_new)).execute()['items'])
                except HttpError:
                    break
                
            try:
                next_page_token = search_response['nextPageToken']
            except KeyError:
                INIT_REQ = True
                print('Last page. {} items collected.'.format(len(search_items)))
        
        print('Collecting complete. {} items collected.'.format(len(search_items)))
                        
        return search_items, video_items
    
    def get_video_info(search_items, video_items):
        result_json = {}
        idx = 0
        for item in search_items:
            if item['id']['kind'] == 'youtube#video':
                vid = item['id']['videoId']
                vitems = [item for item in video_items if item['id'] == vid]
                if vitems:
                    vitem = vitems[0]
                else:
                    continue
                if vitem['contentDetails']['caption']:
                    captions = get_captions(item['id']['videoId'])
                if captions:
                    result_json[idx] = info_to_dict(item['id']['videoId'], item['snippet']['title'], item['snippet']['description'], vitem['statistics']['viewCount'], captions)
                    idx += 1
        return result_json
    
    def info_to_dict(videoId, title, description, viewCount, captions):
        result = {
            "videoId": videoId,
            "title": title,
            "description": description,
            "viewCount": viewCount,
            "captions": captions
        }
        return result
    
    def get_captions(vid):
        try:
            transcript_list = YouTubeTranscriptApi.list_transcripts(vid)
        except:
            return ''
    
        for transcript in transcript_list:
            if transcript.language_code == 'en':
                try:
                    caption_script = ' '.join([t.get('text') for t in transcript.fetch()])
                except Exception as e:
                    print(e)
                    return ''
                continue
            if transcript.is_translatable:
                try:
                    caption_script = ' '.join([t.get('text') for t in transcript.translate('en').fetch()])
                except Exception as e:
                    print(e)
                    return ''
            
        return caption_script
    ```
    

## 수집된 데이터

- apple iphone 15: 696건
    - 예시
        
        ```
        videoId:"w3-yM1IjuB0"
        title:"iPhone 15 First Look: Why Apple Switched to USB-C Ports | WSJ"
        description:"Sure, Apple's new iPhone 15 features have improved designs and cameras, but the tech company's switch from the Lightning to ..."
        viewCount:"325360"
        captions:"here they are everyone the new iPhone 15 and iPhone 15 Pro Models the new iPhone 15 Pros have a new titanium design action button and improved cameras but the biggest news from today's Apple event we're bringing USBC to iPhone 15. yep I basically came all the way to Apple's Cupertino headquarters to see a tiny hole and to see if anyone would well take my lightning collection which is basically trash but seriously the switch from lightning to USBC may be the biggest iPhone news to impact you in years allow me to answer all your important questions I won't apologize for that pond number one what's this port anyway there are four new iPhone 15 models and they all have USBC ports it's the same port found on Android phones iPads and most Windows and Mac laptops and pretty much every other modern consumer Gadget so that means you can charge all those things with one type of cable it's goodbye to Apple's favorite proprietary Port introduced in 2012. ..."
        ```
        
- samsung galaxy s23: 901건
    - 예시
        
        ```
        videoId:"lVPLUIPewLc"
        title:"Samsung Galaxy S23 vs S23 Plus vs S23 Ultra - Which should you Buy?"
        description:"Samsung Galaxy S23 vs S23 Plus vs S23 Ultra 5G Hands-On First Look & Impressions ▻BUY Samsung Galaxy S23 Ultra ..."
        viewCount:"833664"
        captions:"- The Samsung Galaxy S23 Series is finally official. I've had some early hands-on time and here's everything you need to know. What's up, guys? Saf here on SuperSaf TV, and as always we're going to be breaking down all of the differences between these devices, SuperSaf Style. So to start off with, as expected, we do have three new Samsung Galaxy S23 devices. There's the S23, the S23 Plus, as well as the S23 Ultra. Now, the S23 Ultra does have a very similar design to the S22 Ultra that we saw last year, with the biggest design difference on the S23 Ultra being that we have a new curvature on the edges, which gives you more flat surface area so it's not as curved as the S22 Ultra. ..."
        ```
        

# 토픽 모델링

- LDA, BERTopic 2가지 방식으로 진행 후 결과 비교
- title, desscription, captions를 공백으로 합쳐 input data로 사용
- 전처리
    - 특수문자, 숫자 제거
    - nltk stopwords 제거
    - query에 포함된 단어 제거
    - lemmatization 수행

## LDA

- num_topics: 토픽 개수. 20개로 설정.
- iterations: LDA 모델 iteration 횟수. 기본값은 50이나 본 실험에서는 500으로 설정.
    

## BERTopic

- min_topic_size: 토픽의 최소 사이즈(토픽에 해당하는 문서 수). 기본값은 10이나 5로 설정.
    - 초기 실험 시 토픽이 3개로 산출되어 토픽 수를 늘리기 위해 조정.
    
