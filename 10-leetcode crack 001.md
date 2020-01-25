### 茴香豆的茴字有几种写法 之 "Two Sum"

这个系列用来总结和整理leetcode的算法题目，我的原则如下：
1.我只会整理leetcode中的“赞”比“踩”多的题目，因为这类题目描述清晰，测试用例也比较合理，不会有太tricky的情况。我基本是按照编号顺序整理。
2.每道题目的代码会有python和c++两种，来源大都是leetcode的讨论区。

今天先从第一题开始： https://leetcode.com/problems/two-sum/submissions/

这道题应该是每个码农必刷的题目了，但是并不是看上去那么简单，比如如果只能对所有元素遍历一次的话，求解？哈哈，可以参考下面的代码。

Python的解法：
https://leetcode.com/problems/two-sum/discuss/17/Here-is-a-Python-solution-in-O(n)-time
class Solution(object):
    def twoSum(self, nums, target):
        buff_dict = {}
        for i in range(len(nums)):
            if nums[i] in buff_dict:
                return [buff_dict[nums[i]], i]
            else:
                buff_dict[target - nums[i]] = i

C++的解法：
class Solution {
public:
vector<int> twoSum(vector<int>& nums, int target) {
map<int, int> dict;
for (int i = 0; i<nums.size(); ++i) {
if (dict.find(nums[i]) != dict.end())
return { dict[nums[i]], i };
else
dict[target - nums[i]] = i;
}

return vector<int>();
}
};
