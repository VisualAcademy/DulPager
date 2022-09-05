# DulPager
Blazor Pager Component


## 블레이저 페이저 컴포넌트

블레이저의 모든 영역에서 공통으로 사용할 수 있는 페이저(페이징) 컴포넌트 소스입니다.

DulPager 이름으로 NuGet 패키지에 공개되어 있습니다.

## DulPager가 적용된 프로젝트들

https://github.com/VisualAcademy/Hawaso

https://github.com/VisualAcademy/ReplyApp

https://github.com/VisualAcademy/UploadApp

기타

## 게시판 페이징 처리 로직

***DulPagerBase.cs***

```C#
namespace DulPager
{
    /// <summary>
    /// 페이저를 위한 공통 클래스
    /// </summary>
    public class DulPagerBase
    {
        /// <summary>
        /// 페이저가 사용되는 페이지
        /// </summary>
        public string Url { get; set; }

        /// <summary>
        /// 총 몇 개의 페이지가 만들어지는지: 총 레코드 수 / 10(한 페이지에서 보여줄) 
        /// </summary>
        public int PageCount { get; set; } = 5;

        /// <summary>
        /// 레코드 카운트: 현재 테이블에 몇 개의 레코드가 있는지 지정
        /// </summary>
        public int RecordCount { get; set; } = 50;

        /// <summary>
        /// 페이지 사이즈: 한 페이지에 몇 개의 레코드를 보여줄건지 결정 
        /// </summary>
        public int PageSize { get; set; } = 10;

        /// <summary>
        /// 페이지 인덱스: 현재 보여줄 페이지 번호의 인덱스(PageNumber - 1)
        /// </summary>
        public int PageIndex { get; set; } = 0;

        /// <summary>
        /// 페이지 번호: 현재 보여줄 페이지 번호: 1 페이지, 2 페이지, ... 
        /// </summary>
        public int PageNumber { get; set; } = 1; // 1로 초기화 

        /// <summary>
        /// 페이저에 몇 개씩 페이지 버튼을 표시할지
        /// </summary>
        public int PagerButtonCount { get; set; } = 3;

        #region 검색 리스트 관련 속성들
        /// <summary>
        /// 기본 리스트면 false, 검색 결과에 대한 페이징 리스트면 true
        /// </summary>
        public bool SearchMode { get; set; } = false;

        /// <summary>
        /// 검색할 필드: Name, Title, Content
        /// </summary>
        public string SearchField { get; set; } = "";

        /// <summary>
        /// 검색할 내용
        /// </summary>
        public string SearchQuery { get; set; } = "";
        #endregion
    }
}
```

***DulPagerComponent.razor*** 

```razor
<div class="d-flex">

    <ul class="pagination pagination-sm mx-auto">
        @*처음 링크*@
        @if (Model.PageNumber == 1)
        {
            <li class="page-item">
                <a class="page-link first btn disabled"><span style="font-size: 7pt;">FIRST</span></a>
            </li>
        }
        else
        {
            <li class="page-item">
                <a class="page-link first btn" @onclick="@(() => PagerButtonClicked(1))"><span style="font-size: 7pt;">FIRST</span></a>
            </li>
        }

        @*이전 n개 링크*@
        @if (Model.PageNumber > Model.PagerButtonCount)
        {
            int prev = (Model.PageNumber - 1) / (int)Model.PagerButtonCount * Model.PagerButtonCount; // 이전 n개 페이지 번호 계산
            <li class="page-item">
                <a class="page-link prev btn" @onclick="@(() => PagerButtonClicked(prev))"><span style="font-size: 7pt;">-@Model.PagerButtonCount</span></a>
            </li>
        }
        else
        {
            <li class="page-item">
                <a class="page-link prev btn disabled"><span style="font-size: 7pt;">-@Model.PagerButtonCount</span></a>
            </li>
        }

        @*이전 링크: 처음 페이지가 아니면 페이지 번호를 -1씩 감소*@
        @if (Model.PageNumber > 1)
        {
            int prev = Model.PageNumber - 1; // 이전 페이지 번호 계산
            <li class="page-item">
                <a class="page-link prev btn" @onclick="@(() => PagerButtonClicked(prev))"><span style="font-size: 7pt;">PREV</span></a>
            </li>
        }
        else
        {
            <li class="page-item">
                <a class="page-link prev btn disabled"><span style="font-size: 7pt;">PREV</span></a>
            </li>
        }

        @*페이지 수만큼 숫자 버튼 출력*@
        @{
            int i = 0; // 숫자 버튼, 다음 n개에서 사용
            int start = Model.PageIndex / (int)Model.PagerButtonCount * Model.PagerButtonCount + 1; //[?]
            int end = (Model.PageIndex / (int)Model.PagerButtonCount + 1) * Model.PagerButtonCount; //[?]
        }

        @for (i = start; i <= end; i++)
        {
            var currentNumber = i; // 현재 페이지 번호 임시 저장

            // 페이지 수보다 크면 종료
            if (i > Model.PageCount)
            {
                break;
            }

            // 현재 보고있는 페이지면 링크 제거
            if (i == Model.PageNumber)
            {
                <li class="page-item active">
                    <a class="page-link current btn disabled"><span style="font-size: 7pt;">@i</span></a>
                </li>
            }
            else
            {
                <li class="page-item">
                    <a class="page-link current btn" @onclick="@(() => PagerButtonClicked(currentNumber))"><span style="font-size: 7pt;">@i</span></a>
                </li>
            }
        }

        @*다음 링크: 마지막 페이지가 아니면 페이지 번호를 +1씩 증가*@
        @if (Model.PageNumber < Model.PageCount)
        {
            int next = Model.PageNumber + 1; // 다음 페이지 번호 계산
            <li class="page-item">
                <a class="page-link next btn" @onclick="@(() => PagerButtonClicked(next))"><span style="font-size: 7pt;">NEXT</span></a>
            </li>
        }
        else
        {
            <li class="page-item">
                <a class="page-link next btn disabled"><span style="font-size: 7pt;">NEXT</span></a>
            </li>
        }

        @*다음 n개 링크*@
        @if (i <= Model.PageCount)
        {
            // 다음 n개 페이지 번호 계산
            int next = Model.PageIndex / (int)Model.PagerButtonCount * Model.PagerButtonCount + Model.PagerButtonCount + 1; //[?]
            <li class="page-item">
                <a class="page-link next btn" @onclick="@(() => PagerButtonClicked(next))"><span style="font-size: 7pt;">@(Model.PagerButtonCount)+</span></a>
            </li>
        }
        else
        {
            <li class="page-item">
                <a class="page-link next btn disabled"><span style="font-size: 7pt;">@(Model.PagerButtonCount)+</span></a>
            </li>
        }

        @*마지막 링크*@
        @if (Model.PageNumber != Model.PageCount)
        {
            <li class="page-item">
                <a class="page-link last btn" @onclick="@(() => PagerButtonClicked(Model.PageCount))"><span style="font-size: 7pt;">LAST</span></a>
            </li>
        }
        else
        {
            <li class="page-item">
                <a class="page-link last btn disabled"><span style="font-size: 7pt;">LAST</span></a>
            </li>
        }
    </ul>

</div>

@code {
    //[1] 부모로부터 모든 페이저 관련된 정보가 담김 PagerBase 개체 받음
    [Parameter]
    public DulPagerBase Model { get; set; }

    //[2] Parameter가 넘어오면 자동으로 Pager.PageCount 계산
    protected override Task OnParametersSetAsync()
    {
        Model.PageCount = (Model.RecordCount == 0)
            ? 1 // 페이저 버튼은 최소 1개는 표시
            : Convert.ToInt32(Math.Ceiling(Model.RecordCount / (double)Model.PageSize));
        return base.OnParametersSetAsync();
    }

    //[3] 부모의 콜백 메서드를 PageIndexChanged 대리자로 받음
    [Parameter]
    public EventCallback<int> PageIndexChanged { get; set; }

    //[4] Pager 버튼 클릭할 때 현재 클릭한 페이지 번호를 받아서 부모에게 전달
    private void PagerButtonClicked(int pageNumber)
    {
        Model.PageNumber = pageNumber;
        Model.PageIndex = pageNumber - 1;

        PageIndexChanged.InvokeAsync(pageNumber - 1); // PageIndex를 부모 컴포넌트로 전송
    }
}
```
