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

            //item.style.display = sizes.some(size => sizeFilters.includes(size)) ? "" : "none";

            // 각 사이즈에 대해 필터 조건 중 하나라도 포함되어 있는지 확인
            item.style.display = sizes.some(size => 
                sizeFilters.some(filter => size.toUpperCase().includes(filter.toUpperCase()))
            ) ? "" : "none";
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
    
    // 검색 버튼 클릭 시 새 창에서 URL 열기 기능 추가
    const modifyButtonsForNewWindow = () => {
        // 모든 상품 검색 버튼에 이벤트 추가
        const buttons = document.querySelectorAll("#goodsItemList > li > div > div > div > button");
        buttons.forEach(button => {
            const originalOnclick = button.getAttribute("onclick");
            if (originalOnclick) {
                // onclick 속성에서 상품 정보 추출
                const goodsNoMatch = originalOnclick.match(/goods_no\s*:\s*'([^']+)'/);
                const saleShopDiviCdMatch = originalOnclick.match(/sale_shop_divi_cd\s*:\s*'([^']+)'/);
                const saleShopNoMatch = originalOnclick.match(/sale_shop_no\s*:\s*'([^']+)'/);
                const contsFormCdMatch = originalOnclick.match(/conts_form_cd\s*:\s*'([^']+)'/);
                const contsDistNoMatch = originalOnclick.match(/conts_dist_no\s*:\s*'([^']+)'/);
                const contsDiviCdMatch = originalOnclick.match(/conts_divi_cd\s*:\s*'([^']+)'/);
                const relNoMatch = originalOnclick.match(/rel_no\s*:\s*'([^']+)'/);
                const relDiviCdMatch = originalOnclick.match(/rel_divi_cd\s*:\s*'([^']+)'/);
                
                if (goodsNoMatch && saleShopDiviCdMatch && saleShopNoMatch) {
                    const goodsNo = goodsNoMatch[1];
                    const saleShopDiviCd = saleShopDiviCdMatch[1];
                    const saleShopNo = saleShopNoMatch[1];
                    const contsFormCd = contsFormCdMatch ? contsFormCdMatch[1] : '100';
                    const contsDistNo = contsDistNoMatch ? contsDistNoMatch[1] : goodsNo;
                    const contsDiviCd = contsDiviCdMatch ? contsDiviCdMatch[1] : '20';
                    const relNo = relNoMatch ? relNoMatch[1] : goodsNo;
                    const relDiviCd = relDiviCdMatch ? relDiviCdMatch[1] : '10';
                    
                    // URL 생성
                    const url = `https://www.sivillage.com/goods/initDetailGoods.siv?goods_no=${goodsNo}&sale_shop_divi_cd=${saleShopDiviCd}&sale_shop_no=${saleShopNo}&conts_form_cd=${contsFormCd}&conts_dist_no=${contsDistNo}&conts_divi_cd=${contsDiviCd}&rel_no=${relNo}&rel_divi_cd=${relDiviCd}`;
                    
                    // 새 이벤트 핸들러 설정
                    button.onclick = (e) => {
                        e.preventDefault();
                        e.stopPropagation();
                        window.open(url, '_blank');
                        return false;
                    };
                    
                    // 버튼 스타일 변경하여 수정되었음을 표시
                    button.style.backgroundColor = "#3498db";
                    button.style.color = "white";
                }
            }
        });
        
        // 상품 링크(a 태그)에 대한 처리
        const productLinks = document.querySelectorAll("#goodsItemList > li > div > a");
        productLinks.forEach(link => {
            const li = link.closest('li');
            if (!li) return;
            
            const goodsNo = li.getAttribute("data-goods_no");
            if (!goodsNo) return;
            
            // 기존 이벤트 방지하고 새 창 열기
            link.addEventListener('click', (e) => {
                e.preventDefault();
                e.stopPropagation();
                
                // 필요한 파라미터들을 찾아서 URL 구성
                // 버튼과 동일한 URL 파라미터 구조 사용
                const url = `https://www.sivillage.com/goods/initDetailGoods.siv?goods_no=${goodsNo}&sale_shop_divi_cd=12&sale_shop_no=2502169167&conts_form_cd=100&conts_dist_no=${goodsNo}&conts_divi_cd=20&rel_no=${goodsNo}&rel_divi_cd=10`;
                
                window.open(url, '_blank');
                return false;
            }, true);
            
            // 수정된 링크 스타일 변경
            link.style.textDecoration = "underline";
            link.style.textDecorationColor = "#3498db";
        });
        
        console.log("모든 검색 버튼과 상품 링크가 새 창에서 열리도록 수정되었습니다.");
    };
    
    // 버튼 컨테이너 생성
    const buttonContainer = document.createElement("div");
    buttonContainer.style.position = "fixed";
    buttonContainer.style.left = "20px";
    buttonContainer.style.top = "150px";
    buttonContainer.style.width = "300px";
    buttonContainer.style.display = "flex";
    buttonContainer.style.flexDirection = "column";
    buttonContainer.style.zIndex = "1000";
    buttonContainer.style.boxShadow = "2px 2px 10px rgba(0,0,0,0.2)";
    
    // 타이틀 추가 및 드래그 기능 설정
    const titleDiv = document.createElement("div");
    titleDiv.textContent = "Shopping Helper";
    titleDiv.style.width = "100%";
    titleDiv.style.height = "16px";
    titleDiv.style.backgroundColor = "#f8f8f8";
    titleDiv.style.fontWeight = "bold";
    titleDiv.style.textAlign = "center";
    titleDiv.style.padding = "5px 0";
    titleDiv.style.borderRadius = "5px 5px 0 0";
    titleDiv.style.borderBottom = "1px solid #ccc";
    titleDiv.style.cursor = "move";

    // 드래그 관련 변수
    let isDragging = false;
    let currentX;
    let currentY;
    let initialX;
    let initialY;

    // 마우스 이벤트 핸들러
    titleDiv.addEventListener('mousedown', (e) => {
        isDragging = true;
        initialX = e.clientX - buttonContainer.offsetLeft;
        initialY = e.clientY - buttonContainer.offsetTop;
    });

    document.addEventListener('mousemove', (e) => {
        if (!isDragging) return;
        
        e.preventDefault();
        currentX = e.clientX - initialX;
        currentY = e.clientY - initialY;

        // 화면 경계 체크
        const maxX = window.innerWidth - buttonContainer.offsetWidth;
        const maxY = window.innerHeight - buttonContainer.offsetHeight;
        
        currentX = Math.min(Math.max(0, currentX), maxX);
        currentY = Math.min(Math.max(0, currentY), maxY);

        buttonContainer.style.left = `${currentX}px`;
        buttonContainer.style.top = `${currentY}px`;
    });

    document.addEventListener('mouseup', () => {
        isDragging = false;
    });
    
    // 버튼들을 담을 컨테이너 생성
    const buttonsWrapper = document.createElement("div");
    buttonsWrapper.style.display = "flex";
    
    // 사이즈 필터 입력창 생성 (120px로 확장)
    const sizeInput = document.createElement("input");
    sizeInput.type = "text";
    sizeInput.value = "L,XL,XXL,52,54";
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
    
    // 링크 수정 버튼 생성
    const modifyLinksButton = document.createElement("button");
    modifyLinksButton.textContent = "새창";
    modifyLinksButton.style.flex = "3";
    modifyLinksButton.style.padding = "10px";
    modifyLinksButton.style.backgroundColor = "#3498db";
    modifyLinksButton.style.color = "#ffffff";
    modifyLinksButton.style.border = "none";
    modifyLinksButton.style.borderRight = "1px solid #ffffff";
    modifyLinksButton.style.cursor = "pointer";
    modifyLinksButton.style.fontWeight = "bold";
    modifyLinksButton.onclick = modifyButtonsForNewWindow;
    
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
    
    // 버튼들을 새로운 wrapper에 추가
    buttonsWrapper.appendChild(sizeInput);
    buttonsWrapper.appendChild(modifyLinksButton);
    buttonsWrapper.appendChild(reverseButton);
    buttonsWrapper.appendChild(sortButton);

    // 타이틀과 버튼들을 메인 컨테이너에 추가
    buttonContainer.appendChild(titleDiv);
    buttonContainer.appendChild(buttonsWrapper);
    
    document.body.appendChild(buttonContainer);
    
    // 페이지 로드 시 자동으로 링크 수정 실행
    setTimeout(() => {
        modifyButtonsForNewWindow();
    }, 1000);
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
