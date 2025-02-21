# ShoppingHelper
쇼핑을 간편하게..

# Use JavaScript & Css Plugin

> script side

```
var startReady = () => {
    // 버튼 클릭 시 항상 최신 ul을 가져오도록 변경
    const getUl = () => document.getElementById("goodsItemList");
    
    // 가격 가져오는 함수 정의
    const getPrice = (li) => {
        const priceElement = li.querySelector(".product__data-price");
        if (!priceElement) return 0;
        
        const priceSpans = priceElement.getElementsByTagName("span");
        if (priceSpans.length < 2) return 0;
        
        const priceText = priceSpans[1].textContent.replace(/[^0-9]/g, "");
        return parseInt(priceText, 10) || 0;
    };
    
    // 상품의 사이즈 가져오기 (이미 로드된 경우 또는 AJAX 요청 필요 여부 확인)
    const getSizes = async (li) => {
        const sizeContainer = li.querySelector(".siv-product__size-list");
        if (sizeContainer && sizeContainer.children.length > 0) {
            return Array.from(sizeContainer.getElementsByTagName("span")).map(span => span.textContent.trim());
        }
        
        const goodsNo = li.getAttribute("data-goods_no");
        if (!goodsNo) return [];
        
        try {
            const response = await fetch(`/dispctg/searchGoodsItemListAjax.siv?goods_no=${goodsNo}`);
            const data = await response.json();
            return data.itemList.map(item => item.OPT_VAL_NM2.trim());
        } catch (error) {
            console.error("사이즈 정보를 불러오는 데 실패했습니다.", error);
            return [];
        }
    };
    
    // 필터링 함수 정의 (콤마로 구분된 다중 사이즈 지원)
    const filterBySize = async (sizeInput) => {
        const ul = getUl();
        if (!ul) return;
        
        const sizeFilters = sizeInput.split(",").map(s => s.trim()).filter(Boolean);
        const items = Array.from(ul.getElementsByClassName("product__item"));
        
        for (let item of items) {
            const sizes = await getSizes(item);
            item.style.display = sizes.some(size => sizeFilters.includes(size)) ? "" : "none";
        }
    };
    
    // 정렬 함수 정의 (오름차순)
    const sortAscending = () => {
        console.log("정렬 버튼 클릭됨");
        const ul = getUl();
        if (!ul) {
            console.error("상품 목록을 찾을 수 없습니다.");
            return;
        }
        let items = Array.from(ul.getElementsByClassName("product__item"));
        
        items.sort((a, b) => getPrice(a) - getPrice(b));
        
        items.forEach(item => ul.appendChild(item));
        console.log("상품이 가격 기준으로 오름차순 정렬되었습니다.");
    };
    
    // 정렬 함수 정의 (내림차순)
    const sortDescending = () => {
        console.log("역정렬 버튼 클릭됨");
        const ul = getUl();
        if (!ul) {
            console.error("상품 목록을 찾을 수 없습니다.");
            return;
        }
        let items = Array.from(ul.getElementsByClassName("product__item"));
        
        items.sort((a, b) => getPrice(b) - getPrice(a));
        
        items.forEach(item => ul.appendChild(item));
        console.log("상품이 가격 기준으로 내림차순 정렬되었습니다.");
    };
    
    // 버튼 컨테이너 생성
    const buttonContainer = document.createElement("div");
    buttonContainer.style.position = "fixed";
    buttonContainer.style.top = "150px";
    buttonContainer.style.right = "20px";
    buttonContainer.style.width = "300px";
    buttonContainer.style.display = "flex";
    buttonContainer.style.zIndex = "1000";
    buttonContainer.style.boxShadow = "2px 2px 10px rgba(0,0,0,0.2)";
    
    // 사이즈 필터 입력창 생성 (120px로 확장)
    const sizeInput = document.createElement("input");
    sizeInput.type = "text";
    sizeInput.placeholder = "Size (e.g. S,M,L)";
    sizeInput.style.width = "120px";
    sizeInput.style.textAlign = "center";
    sizeInput.style.border = "1px solid #ccc";
    sizeInput.style.borderRadius = "5px 0 0 5px";
    sizeInput.style.padding = "5px";
    sizeInput.onkeypress = (event) => {
        if (event.key === "Enter") {
            filterBySize(sizeInput.value.trim());
        }
    };
    
    // 역정렬 버튼 생성 및 스타일 적용 (왼쪽 30%)
    const reverseButton = document.createElement("button");
    reverseButton.textContent = "역";
    reverseButton.style.flex = "3";
    reverseButton.style.padding = "10px";
    reverseButton.style.backgroundColor = "#444444";
    reverseButton.style.color = "#ffffff";
    reverseButton.style.border = "none";
    reverseButton.style.borderRight = "1px solid #ffffff";
    reverseButton.style.cursor = "pointer";
    reverseButton.style.fontWeight = "bold";
    reverseButton.onclick = sortDescending;
    
    // 정렬 버튼 생성 및 스타일 적용 (오른쪽 70%)
    const sortButton = document.createElement("button");
    sortButton.textContent = "정렬";
    sortButton.style.flex = "7";
    sortButton.style.padding = "10px";
    sortButton.style.backgroundColor = "#ff5733";
    sortButton.style.color = "#ffffff";
    sortButton.style.border = "none";
    sortButton.style.borderRadius = "0 5px 5px 0";
    sortButton.style.cursor = "pointer";
    sortButton.style.fontWeight = "bold";
    sortButton.onclick = sortAscending;
    
    buttonContainer.appendChild(sizeInput);
    buttonContainer.appendChild(reverseButton);
    buttonContainer.appendChild(sortButton);
    document.body.appendChild(buttonContainer);
};

setTimeout(function(){
    startReady();
}, 2000);
```

> CSS side

```
.siv-product__size {
    display: block !important;
}
```
