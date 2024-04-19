<!DOCTYPE html>
<html>
<body>
    <h2 style="text-align: center;">ASL Dictionary</h2>
   
   
    
</body>

<footer class="footer">
    <div class="footer-copyright">
        <div class="container">
        <p style="font-size:0.8vw">Credits: <a href="http://www.islrtc.nic.in/isl-dictionary-launch" target="_blank">TOP CHANNEL</a> & <a href="https://indiansignlanguage.org/" target="_blank">LEARN LANGUAGE CENTER</a> & <a href="https://winvinayafoundation.org/" target="_blank">LANGUAGE SMIP</a><br>
        </div>
    </div>
</footer>

<script>
        $(document).ready(function () {

            videos_data_url = "https://raw.githubusercontent.com/AI4Bharat/sign-language.ai4bharat.org/master/data/all_videos.json"
            
            $('iframe').each(function() {
                    if ($(this).attr('src') == '') {
                        $(this).hide();
                    }
                });

            const options = {
                isCaseSensitive: false,
                shouldSort: true,
                ignoreLocation: true,
                // includeScore: true,
                keys: [
                    "Title",
                ]
            };

            $.getJSON(videos_data_url, function (data) {
                const fuse = new Fuse(data, options);

                function rerank(search_term, search_results, max_terms=5) {
                    let final_results = [];
                    let exact_match_indices = new Set();
                    const partial_pattern = new RegExp(search_term, 'gi');
                    const exact_pattern = new RegExp("([\\W]|^)" + search_term + "([\\W]|$)", 'gi');
                    for (i = 0; i < search_results.length; i++) {
                        title = search_results[i].item.Title;
                        if (!title.match(partial_pattern)) {
                            break;
                        }

                        if (title.match(exact_pattern)) {
                            final_results.push(search_results[i]);
                            exact_match_indices.add(i);
                            if (final_results.length >= max_terms) {
                                return final_results;
                            }
                        }
                    }

                    if (exact_match_indices.size > 0) {
                        for (i = 0; i < search_results.length; i++) {
                            if (exact_match_indices.has(i)) {
                                continue;
                            }
                            final_results.push(search_results[i]);
                            if (final_results.length >= max_terms) {
                                break;
                            }
                        }
                        return final_results;
                    } else {
                        return search_results.slice(0, max_terms);
                    }
                }

                $('#dict_searchbar').keyup(function () {
                    $('#content').hide();
                    const search_term = $(this).val();
                    let result = fuse.search(search_term);
                    let resultdiv = $('#dict_result');

                    if (result.length === 0) {
                        resultdiv.hide();
                    }
                    else {
                        resultdiv.empty();
                        result = rerank(search_term, result, 5);
                        for (i=0; i<result.length; i++) {
                            let searchitem = "";
                            if(result[i].item.Domain) {
                                searchitem = '<li><a href=' + result[i].item.URL + ' >' + result[i].item.Title + ' (' +result[i].item.Domain + ')' + ' -- by ' + result[i].item.Source +'</a></li>';
                            }
                            else
                            {
                                searchitem = '<li><a href=' + result[i].item.URL + ' >' + result[i].item.Title + ' -- by ' + result[i].item.Source + '</a></li>';
                            }
                            resultdiv.append(searchitem);
                        }
                        resultdiv.show();
                    }
                });
                
                var $content = $('#content');
                $(document).on("click", "#dict_result li a" , function() {
                    $content.show();
                    $content.attr('src', $(this).attr('href').replace("watch?v=","embed/"));
                    return false;
                });

            });

        });

        

</script>

</html>
